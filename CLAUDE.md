# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start dev server
npm run build    # Production build
npm run lint     # Run Next.js linter
npm start        # Start production server
```

No test suite is configured.

## Architecture

**Next.js 14 App Router** personal portfolio with TypeScript, Tailwind CSS, Turso (libSQL) database, Pusher real-time presence, and a protected admin dashboard.

### Key Directories

- `app/` — Pages, layouts, and API routes (App Router)
- `app/admin/` — Password-protected admin dashboard (force-dynamic server components)
- `app/api/` — REST API endpoints
- `components/` — Reusable React components
- `lib/` — Utility modules (db, auth, logging, journal, pusher, etc.)

### Navigation & View Model

The portfolio uses a hybrid routing model. `LeftNav` (client component) controls a `PortfolioView` state (`'home' | 'journal'`) that switches between sections in-place without full navigation. The `?tab=journal` query param supports deep linking. Admin pages are a hard navigation away.

### Database (lib/db.ts)

Turso (libSQL) singleton. All CRUD is in one file. Tables are auto-created via `initDb()` / `initJournalDb()` which use `CREATE TABLE IF NOT EXISTS` and `ALTER TABLE IF NOT EXISTS` for zero-downtime migrations. No ORM — raw SQL.

### Authentication

- **Admin session:** iron-session cookie (`portfolio_admin_session`), AES-256, 24h TTL
- **Password:** bcrypt (12 rounds) stored in `admin_config` table
- **Route guard:** `middleware.ts` redirects unauthenticated `/admin/*` to `/admin/login`

### Server vs. Client Components

- **Server components:** Page layouts, data fetching, admin dashboard data loads
- **Client components (`'use client'`):** Interactivity — modals, forms, tabs, tables, reactions, Pusher

### Real-Time Presence (Pusher)

All visitors join `presence-portfolio`. Mouse moves emit `client-cursor-moved` events (client-triggered, throttled to 50ms). Cursor positions are normalized to viewport fractions (0–1). Auth via `/api/pusher/auth`.

### Logging (lib/logger.ts)

Structured JSON to stdout/stderr captured by Vercel logs. Debug logs are suppressed in production.

```typescript
logger.info('context', 'message', data)
logger.error('context', 'message', error)
```

### Tailwind Theme

Custom dark theme color palette defined in `tailwind.config.ts`: `bg`, `surface`, `accent`, `border`, `text`, `muted`. Use these tokens instead of raw Tailwind colors.

### Environment Variables

See `.env.local.example` for required variables: Turso DB URL + auth token, GitHub token, Pusher credentials, iron-session password.
