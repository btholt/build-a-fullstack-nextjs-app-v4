# Build a Fullstack Next.js App, v4

## Introduction & Setup

- Welcome to Fullstack Next.js
- Thanks to Scott Moss & building on v3
- Prerequisites: Intro to React, Intermediate React, and Intro to Next.js
- The Stack: Our opinionated choices and alternatives
- Project Overview: Building a Team Wiki/Knowledge Base
- Initial setup and project structure

## Styling & UI Foundation

- We're going to style a sign up / sign in page
- Tailwind CSS setup and configuration
- Radix UI primitives and shadcn/ui components
- Building our wiki's core layout and navigation
- Responsive design patterns

## Authentication & Authorization

- Neon Auth setup and configuration
- Middleware for route protection
- User registration and login flows
- Role-Based Access Control (RBAC): Admin, Editor, Viewer
- Protecting wiki pages and admin routes

## Database & Data Layer

- Neon Postgres setup and configuration
- Drizzle ORM setup
- Database schema design (users, pages, revisions)
- Relationships and foreign keys
- CRUD operations for wiki pages
- Database migrations and versioning

## File Storage & Rich Content

- Vercel Blob for file uploads
- Image handling in wiki pages
- File attachment system
- Rich text editing considerations

## Caching & Performance

- Upstash Redis setup
- When and what to cache in a wiki
- Cache invalidation strategies
- Performance optimization patterns

## Email Integration

- Resend setup for transactional emails
- Welcome emails and email verification
- Notification emails for page updates
- Email templates and best practices

## AI Integration

- Setting up AI inference (OpenAI/Anthropic via Vercel AI SDK)
- Page content summarization
- AI-powered search suggestions
- Implementing AI features thoughtfully

## Deployment & Production

- Environment management: local → preview → production
- Neon database branching strategy
- Vercel deployment configuration
- Environment variables and secrets
- Basic monitoring and error tracking

## Observability & Maintenance

- Vercel's built-in analytics and logs
- Error monitoring and alerting
- Performance monitoring
- Maintenance best practices

## Wrap-up

- What we built and what you learned
- Next steps and advanced topics
- Resources for continued learning

# Cut

## Testing

- Playwright setup for end-to-end testing
- Testing authentication flows
- Testing CRUD operations
- Testing deployment readiness

## API Design & Error Handling

- REST API patterns and conventions
- Error handling and user feedback
- Validation with Zod
- API rate limiting
