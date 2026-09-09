# PRD — Personal Dating CRM ("Baddiesforce" internal joke name)

## 1. Summary

A single-user personal CRM for tracking one's own dating life: contacts, dates, notes, and
follow-ups, with calendar sync and voice-memo journaling. All data is self-reported by the
one user about their own experiences — no scraping of other people's messages, socials, or
private accounts, and nothing is shared automatically with anyone else.

The "Baddiesforce" branding, gen-z/corporate tone, and jokes live only in internal docs
(this PRD, PLAN.md, CLAUDE.md, README). The in-app UI copy is neutral and professional
(e.g. "Contacts", "Interactions", not slang).

## 2. Problem Statement

People dating multiple or many people casually lose track of: who they've told what,
where they've already been on a first date, how a person seemed after the last date, and
what they said they'd follow up on. This is normal personal organization — the same
category as a personal CRM freelancers use for clients, applied to dating.

## 3. Target User

- Single user, tracking their own dating life.
- No feature in this app operates on another person's private data without that data
  having been given directly to the user (e.g., the user's own memory of a date, a
  calendar invite the user created, a voice memo the user recorded of their own
  reflections).

## 4. Goals

- Reduce mental overhead of tracking multiple dating relationships.
- Make it easy to log a date and its outcome in under a minute (via voice memo).
- Surface useful summaries: upcoming dates, who you haven't followed up with, priority
  contacts.
- Sync with the user's own calendar so dates show up in one place.

## 5. Core User Stories

1. As the user, I can add a person to my contact list with a name, how we met, and notes.
2. As the user, I can flag a contact as high/medium/low priority, and change that flag
   any time.
3. As the user, I can log a date (manually, or by linking an existing calendar event):
   where, when, and free-text notes.
4. As the user, I can record a voice memo right after a date reflecting on how it went;
   the app transcribes it and extracts a mood tag and any follow-up items I mentioned
   (e.g. "text her about the concert tickets").
5. As the user, I can see a per-contact timeline of all logged interactions.
6. As the user, I can see a dashboard of upcoming dates (pulled from my calendar) and
   open follow-up items across all contacts.
7. As the user, when I log a new date, the app can create a corresponding event on my
   own Google Calendar.

## 6. Key Features

| Feature | Description |
|---|---|
| Contacts / Rolodex | CRUD for people: name, notes, tags, priority flag |
| Interaction log | Dated entries per contact: location, notes, mood |
| Calendar sync (read) | Pull the user's own calendar to show upcoming dates |
| Calendar sync (write) | Create calendar events from logged dates |
| Voice memo capture | Record/upload audio tied to a contact or interaction |
| Transcription + extraction | Turn audio into text, pull out mood + follow-up items |
| Priority flagging | Manual high/medium/low flag per contact, sortable |
| Dashboard | Upcoming dates + outstanding follow-ups, at a glance |

## 7. Data Privacy Principles

- All data is local/self-owned (SQLite file, or user's own cloud DB instance).
- No data about a contact is ever transmitted to a third party except:
  - Calendar events to the user's own connected Google Calendar.
  - Audio to a transcription API (OpenAI Whisper) for processing — no long-term
    third-party storage beyond the API call.
- No contact is ever notified that they exist in the app.
- No feature exports or sends data about a contact to another person or channel.

## 8. Success Metrics (personal-use, informal)

- Time to log a date + memo: < 60 seconds.
- Contacts dashboard reduces "did I already tell her that" mistakes (qualitative).
- Calendar sync accuracy: 100% of logged dates appear on calendar when enabled.

## 9. Out-of-Scope Future Ideas (not building now)

- iOS/Android native app.
- Multi-device sync beyond a single DB.
- Anything involving another person's accounts or messages
