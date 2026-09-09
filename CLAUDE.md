# CLAUDE.md

Guidance for any Claude Code session working in this repo.

## What this project is

A personal CRM for tracking one's own dating life — contacts, dates, notes,
follow-ups — with Google Calendar sync and voice-memo transcription. See `PRD.md` for
full product scope and `PLAN.md` for the technical build plan and phased milestones.

The internal codename "Baddiesforce" and any gen-z/corporate joke tone belong **only** in
docs (this file, PRD.md, PLAN.md, README.md). In-app UI copy must stay neutral and
professional: "Contacts," "Interactions," "Follow-ups" — not slang.

## Stack

- Next.js 14+ (App Router), TypeScript, Tailwind
- Prisma + SQLite (local-first)
- NextAuth.js (Google provider) for Calendar OAuth
- Google Calendar API (`googleapis`) — read + write scopes
- OpenAI Whisper API for audio transcription
- LLM call (strict JSON output) for mood/follow-up extraction from transcripts

## Repo conventions

- App code under `app/`, using the Next.js App Router file conventions.
- Prisma schema at `prisma/schema.prisma`; run `npx prisma migrate dev` after schema
  changes, don't hand-edit the SQLite file.
- API routes under `app/api/*/route.ts`.
- Keep server-only secrets (API keys, OAuth secrets) in `.env.local`, never commit them,
  never log them.
- Uploaded audio files go to a local `/uploads` directory in dev; don't commit this
  directory (add to `.gitignore`).

## Commands

```
npm run dev              # start dev server
npx prisma migrate dev   # apply schema changes
npx prisma studio        # inspect local DB
npm run lint             # lint
npm run build            # production build
```

## Working style for this repo

- Follow the phased plan in `PLAN.md` — implement one phase at a time, don't jump ahead
  to Calendar/Voice features before core CRUD (Phase 1) works and is tested.
- When adding a schema field, update `PLAN.md`'s data model section to match.
- Prefer server components and server actions over client-side fetch where the App
  Router supports it; keep client components limited to interactive bits (forms, audio
  recorder, priority toggle).
- Extraction from transcripts (mood/follow-ups) is a **suggestion**, not authoritative —
  always leave it editable in the UI, never auto-finalize it.
- Handle Whisper/Calendar API failures gracefully — the app should stay usable in
  manual-entry mode if either integration is down or unauthenticated.
