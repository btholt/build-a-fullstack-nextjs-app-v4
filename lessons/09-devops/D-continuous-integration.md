We can deliver our code to production, and we can monitor it once it's there, but how do we ensure code quality and prevent issues before we even have them?

Continuous Integration, the CI portion of CI/CD.

We are going to use GitHub Actions for this (as it's a great platform for most use cases) but there's a trillion providers out there that do this.

I've already put this file in there in your repo, but I'm going to code this up with you as it's a doozy. A lot CI can be pretty simple, but this full stack enterprise, let's throw the kitchen sink at it.

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  lint-and-format:
    name: Lint and Format Check
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: "npm"
          cache-dependency-path: package-lock.json

      - name: Install dependencies
        run: npm ci

      - name: Run Biome check (lint + format)
        run: npm run lint

  unit-tests:
    name: Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: "npm"
          cache-dependency-path: package-lock.json

      - name: Get Neon branch name
        id: neon-branch-name
        run: echo "BRANCH_NAME=unit-${{ github.run_id }}" >> "$GITHUB_ENV"

      - name: Get branch expiration date as an env variable (2 weeks from now)
        id: get-expiration-date
        run:
          echo "EXPIRES_AT=$(date -u --date '+14 days' +'%Y-%m-%dT%H:%M:%SZ')"
          >> "$GITHUB_ENV"

      - name: Create Neon Branch
        uses: neondatabase/create-branch-action@v6
        id: create-neon-branch
        with:
          project_id: ${{ secrets.NEON_PROJECT_ID }}
          branch_name: ${{ env.BRANCH_NAME }}
          api_key: ${{ secrets.NEON_API_KEY }}
          expires_at: ${{ env.EXPIRES_AT }}

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm run test
        env:
          DATABASE_URL: ${{ steps.create-neon-branch.outputs.db_url }}
          NODE_ENV: test
          NEON_PROJECT_ID: ${{ secrets.NEON_PROJECT_ID }}
          NEON_API_KEY: ${{ secrets.NEON_API_KEY }}
          NEON_BRANCH_NAME: ${{ env.BRANCH_NAME }}
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          NEXT_PUBLIC_STACK_PROJECT_ID: ${{ secrets.NEXT_PUBLIC_STACK_PROJECT_ID }}
          NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY: ${{ secrets.NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY }}
          STACK_SECRET_SERVER_KEY: ${{ secrets.STACK_SECRET_SERVER_KEY }}
          UPSTASH_REDIS_REST_URL: ${{ secrets.UPSTASH_REDIS_REST_URL }}
          UPSTASH_REDIS_REST_TOKEN: ${{ secrets.UPSTASH_REDIS_REST_TOKEN }}
          BLOB_READ_WRITE_TOKEN: ${{ secrets.BLOB_READ_WRITE_TOKEN }}
          BLOB_BASE_URL: ${{ secrets.BLOB_BASE_URL }}
          RESEND_API_KEY: ${{ secrets.RESEND_API_KEY }}

      - name: Delete Neon Branch
        if: always()
        uses: neondatabase/delete-branch-action@v3
        with:
          project_id: ${{ secrets.NEON_PROJECT_ID }}
          branch: ${{ env.BRANCH_NAME }}
          api_key: ${{ secrets.NEON_API_KEY }}

  e2e-tests:
    name: E2E Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: "npm"
          cache-dependency-path: package-lock.json

      - name: Get Neon branch name
        id: neon-branch-name
        run: echo "BRANCH_NAME=e2e-${{ github.run_id }}" >> "$GITHUB_ENV"

      - name: Get branch expiration date as an env variable (2 weeks from now)
        id: get-expiration-date
        run:
          echo "EXPIRES_AT=$(date -u --date '+14 days' +'%Y-%m-%dT%H:%M:%SZ')"
          >> "$GITHUB_ENV"

      - name: Create Neon Branch
        uses: neondatabase/create-branch-action@v6
        id: create-neon-branch
        with:
          project_id: ${{ secrets.NEON_PROJECT_ID }}
          branch_name: ${{ env.BRANCH_NAME }}
          api_key: ${{ secrets.NEON_API_KEY }}
          expires_at: ${{ env.EXPIRES_AT }}

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium

      - name: Run E2E tests
        run: npm run test:e2e:ci
        env:
          NODE_ENV: test
          DATABASE_URL: ${{ steps.create-neon-branch.outputs.db_url }}
          NEON_PROJECT_ID: ${{ secrets.NEON_PROJECT_ID }}
          NEON_API_KEY: ${{ secrets.NEON_API_KEY }}
          NEON_BRANCH_NAME: ${{ env.BRANCH_NAME }}
          TEST_USER_EMAIL: ${{ secrets.TEST_USER_EMAIL }}
          TEST_USER_PASSWORD: ${{ secrets.TEST_USER_PASSWORD }}
          NEXT_PUBLIC_STACK_PROJECT_ID: ${{ secrets.NEXT_PUBLIC_STACK_PROJECT_ID }}
          NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY: ${{ secrets.NEXT_PUBLIC_STACK_PUBLISHABLE_CLIENT_KEY }}
          STACK_SECRET_SERVER_KEY: ${{ secrets.STACK_SECRET_SERVER_KEY }}
          UPSTASH_REDIS_REST_URL: ${{ secrets.UPSTASH_REDIS_REST_URL }}
          UPSTASH_REDIS_REST_TOKEN: ${{ secrets.UPSTASH_REDIS_REST_TOKEN }}
          BLOB_READ_WRITE_TOKEN: ${{ secrets.BLOB_READ_WRITE_TOKEN }}
          BLOB_BASE_URL: ${{ secrets.BLOB_BASE_URL }}
          RESEND_API_KEY: ${{ secrets.RESEND_API_KEY }}

      - name: Delete Neon Branch
        if: always()
        uses: neondatabase/delete-branch-action@v3
        with:
          project_id: ${{ secrets.NEON_PROJECT_ID }}
          branch: ${{ env.BRANCH_NAME }}
          api_key: ${{ secrets.NEON_API_KEY }}
```

- This is fairly sophisticated. For reference,[ here's the one that deploys this course website][simple-action] to GitHub pages. Basically build and ship.
- In this one, we run it whenever new commits land in main and we also run it on PRs automatically.
- We then split the run into four parts so they can be run on three separate machines and go fast: type check with TypeScript, lint/format with Biome, unit test with Vitest, and E2E test with Playwright.
- In each one we set up Node.js, we install deps, and we run the testing scripts. For Playwright, we do need to build the server as well so it can actually run and have Playwright test it.
- We pass in the variables (that we'll set up shortly in GitHub) so that they can be used for testing purposes. These will come from secret variables we set up.
- We could go through and split hairs about what is a variable and what is a secret, but I'm just making everything a secret.

Let's go to the repo, to settings, to secrets and variables, and add all of our _test_ variables into here. You have to do it one-by-one which is a bit of a pain, but once you've done that, you're ready to run your CI checks! You may find you need to fix some Biome errors. But now we can't merge anything that doesn't pass type checking, linting, formatting, unit tests, and E2E tests. That's a good first step of assurance that bad stuff isn't going out.

Okay, now that you have and it's committed, push it to the repo and you should see it start running on every commit and PR. E2E tests take quite a bit longer to run because they set up Chromium every time.

Hooray! Code quality!

[simple-action]: https://github.com/btholt/build-a-fullstack-nextjs-app-v4/blob/main/.github/workflows/next.yaml
