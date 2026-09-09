# PLAN.md — Implementation Plan

## Stack

- **Framework**: Next.js 14+ (App Router), TypeScript
- **DB**: SQLite via Prisma ORM (single file, local-first; swappable for Postgres later)
- **Auth**: NextAuth.js with Google provider (needed for Calendar scopes; single-user but
  keeps OAuth token handling standard)
- **Calendar**: Google Calendar API (`googleapis` npm package), scopes:
  `calendar.readonly` + `calendar.events` (write)
- **Transcription**: OpenAI Whisper API (`whisper-1`) for audio → text
- **Structured extraction**: Single LLM call (Claude or GPT) on the transcript with a
  strict JSON-output prompt to extract `{ mood: string, followUps: string[] }`
- **Styling**: Tailwind CSS
- **Hosting**: Local dev by default (`npm run dev`); optionally deploy to Vercel with a
  hosted Postgres if the user wants remote access — not required for v1

## Data Model (Prisma schema, conceptual)

```prisma
model Person {
  id            String    @id @default(cuid())
  name          String
  metAt         String?
  notes         String?
  priority      Priority  @default(MEDIUM)
  tags          String[]
  createdAt     DateTime  @default(now())
  interactions  Interaction[]
}

enum Priority {
  HIGH
  MEDIUM
  LOW
}

model Interaction {
  id                String    @id @default(cuid())
  personId          String
  person            Person    @relation(fields: [personId], references: [id])
  date              DateTime
  location          String?
  notes             String?
  mood              String?
  followUps         String[]
  calendarEventId   String?
  voiceMemo         VoiceMemo?
  createdAt         DateTime  @default(now())
}

model VoiceMemo {
  id              String       @id @default(cuid())
  interactionId   String       @unique
  interaction     Interaction  @relation(fields: [interactionId], references: [id])
  audioPath       String
  transcript      String?
  processedAt     DateTime?
}
```

## Phased Build Plan

### Phase 0 — Scaffolding

- `create-next-app` with TypeScript + Tailwind + App Router.
- Prisma init with SQLite datasource.
- Basic layout shell, nav: Dashboard / Contacts / Settings.

### Phase 1 — Core CRUD (no integrations)

- Contacts list + detail page.
- Add/edit/delete a Person.
- Priority flag control (segmented control: High/Medium/Low).
- Add/edit an Interaction manually (date, location, notes) under a Person.
- Contact timeline view (interactions sorted desc by date).

### Phase 2 — Dashboard

- "Upcoming" view: interactions with `date >= today`.
- "Open follow-ups" view: aggregate `followUps` arrays across all interactions where not
  marked resolved (add a `resolved: boolean` per follow-up item — adjust schema to a
  join table `FollowUp { id, interactionId, text, resolved }` instead of a raw string
  array for this to work cleanly).

### Phase 3 — Google Calendar integration

- NextAuth + Google OAuth setup, request calendar scopes.
- Read: fetch upcoming events, allow linking an existing event to an Interaction.
- Write: "Add to calendar" button on an Interaction creates a Google Calendar event via
  API and stores the returned `calendarEventId`.

### Phase 4 — Voice memo capture + transcription

- Audio upload/record UI (browser `MediaRecorder` API) tied to an Interaction.
- Store audio file (local `/uploads` dir for v1; S3-compatible bucket later if deployed).
- On upload, call Whisper API → store transcript on `VoiceMemo.transcript`.
- Second call: pass transcript to an LLM with a strict JSON-schema prompt to extract
  `mood` and `followUps[]`; write results back onto the `Interaction`.
- Show transcript + extracted fields on the Interaction detail view, editable by hand
  (extraction is a starting point, not authoritative).

### Phase 5 — Polish

- Search/filter contacts by tag or priority.
- Basic empty states, loading states, error handling on API failures (esp. Whisper/
  Calendar API failures — degrade gracefully, let user fill in manually).

## Environment Variables Needed

```
DATABASE_URL=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
NEXTAUTH_SECRET=
OPENAI_API_KEY=
```

## Milestone Definition of Done

- v1 "usable": Phases 0–2 complete — can add contacts, log dates by hand, see dashboard.
- v1.1 "calendar": Phase 3 complete.
- v1.2 "voice": Phase 4 complete — full core loop from PRD works end to end.
