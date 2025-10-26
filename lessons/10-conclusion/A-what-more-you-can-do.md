And with that, we have fully finished our full stack wiki app! Congrats!

I always like to leave some suggestions of how to tinker around afterwards so you can extend your understanding - we have a very cool app, what could we do?

# Extending Wikimasters - Project Ideas

Here are some ideas for expanding the Wikimasters project to practice concepts you've learned or branch into adjacent areas:

## New Features & Services

### Search & Discovery

- Add full-text search using Postgres's built-in search or Algolia/Typesense
- Implement tag/category system for organizing articles
- Create a "related articles" feature using embeddings and vector search

### Collaboration Features

- Add comments/discussions on articles using something like Commento or build your own
- Implement article revision history (show diffs between versions)
- Add collaborative editing with presence indicators (who's viewing/editing) with something like PartyKit
- Create article templates that users can start from

### Content Enhancement

- Integrate an AI writing assistant (suggest improvements, fix grammar)
- Implement table of contents auto-generation from markdown headings

### Media & Assets

- Add drag-and-drop image uploads with a nice UI with the ability to have multiple images
- Create an image gallery/media library for reusing images
- Implement PDF generation for articles (export to PDF)
- Add Open Graph image generation for social sharing

### Notifications & Communication

- Add real-time notifications using Pusher or Socket.io
- Implement @mentions in articles to notify other users
- Create a daily/weekly digest email of new/updated articles
- Add Slack/Discord webhook integration for team notifications
- Add Twilio for text notifications

### Analytics & Insights

- Track article views and reading time
- Create a dashboard showing popular articles, active authors
- Add activity feeds (recent edits, new articles, trending content)
- Implement article analytics (which sections get read most)

### Quality & Workflow

- Add article drafts vs published state workflow
- Implement peer review/approval process before publishing
- Create article templates or boilerplates
- Add spell-check and grammar checking with LanguageTool API

### Performance & Scale

- Implement incremental static regeneration for popular articles
- Add edge caching with Cloudflare Workers
- Set up database read replicas on Neon for scaling reads
- Implement pagination or infinite scroll for article lists

### Developer Experience

- Create webhooks for article events (published, updated)
- Build a CLI tool for bulk importing markdown files
- Add export functionality (download all your articles as markdown)

### Fun & Experimental

- Add dark mode toggle (practice theme management)
- Implement keyboard shortcuts for power users
- Add voice-to-text for article dictation

## Refactoring & Best Practices

### Code Quality

- Expand test coverage to 80%+ (practice TDD)
- Add Storybook for component documentation
- Implement proper error boundaries throughout the app
- Add loading skeletons for better perceived performance

### Architecture

- Refactor to use a design system (practice component architecture)
- Implement proper logging with Pino or Winston
- Add feature flags with one of the vendors on the Vercel Marketplace
- Set up deployment environments with dev, staging, and prod
- Set up rolling deployments and canary deployments

### Security

- Add two-factor authentication
- Implement content security policy (CSP)
- Add rate limiting to prevent abuse
- Implement audit logs (who did what when)

## Integration Ideas

### External Services

- Integrate with GitHub to sync markdown files
- Add Google Drive import/export
- Connect to Notion API for syncing
- Implement SSO with Okta or Auth0
- Set up proper dev auth by making another Neon project and sync'ing anonymized users from prod into dev

### AI/ML

- Auto-generate article summaries with GPT
- Implement semantic search using embeddings
- Add content moderation with OpenAI Moderation API
- Create AI-powered article recommendations

### Monitoring & Ops

- Set up more sophisticated monitoring with Datadog or New Relic
- Add error tracking with Sentry (beyond basics)
- Implement uptime monitoring with Pingdom
- Create status page for service health
