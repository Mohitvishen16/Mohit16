# CricPulse — Live Cricket Scores & Smart Match Notifications

CricPulse is a cross-platform app concept that gives users:
- **Live ball-by-ball score updates** for ongoing cricket matches.
- **All worldwide matches** across international, domestic, and franchise leagues.
- **Instant notifications** for key events (wickets, milestones, innings breaks, result).

## 1) Product Goals

1. Show *every* scheduled/ongoing cricket match globally in one place.
2. Deliver low-latency score updates (target: under 5–10 seconds from provider update).
3. Provide personalized notifications so users are not spammed.
4. Work well on low-end networks/devices.

## 2) Core Features

### A. Match Discovery
- Global calendar by date/timezone.
- Filters by format: Test, ODI, T20, T10.
- Filters by tournament/series/team.
- Tabs: **Live**, **Upcoming**, **Completed**.

### B. Live Match Center
- Live scorecard (runs/wickets/overs, RR, CRR, required RR).
- Ball-by-ball feed.
- Partnership and batter/bowler mini-stats.
- Match state: toss, innings break, rain delay, result.

### C. Notifications
- Subscribe per team / tournament / specific match.
- Event-level controls:
  - Match start
  - Wicket
  - 50/100 milestones
  - Innings end
  - Chase equation alerts
  - Match result
- Quiet hours + frequency throttle.

### D. User Features
- Anonymous mode (no login) for quick access.
- Optional account for cloud sync of follows and settings.
- Favorite teams and pinned matches.

## 3) Suggested Tech Stack

### Mobile App
- **Flutter** (single codebase for Android + iOS).
- State management: Riverpod.
- Local cache: Hive / SQLite.
- Push notifications: Firebase Cloud Messaging (FCM) + APNs bridge.

### Backend
- **Node.js + NestJS** (or FastAPI as alternative).
- WebSocket gateway for live updates.
- REST APIs for schedules, scorecards, settings.
- Redis for pub/sub + rate-limited notification fanout.
- PostgreSQL for users, subscriptions, preferences.

### Data Source
- Cricket data provider API (commercial/official feed).
- Ingestion worker polls/streams provider and normalizes events.

## 4) High-Level Architecture

1. **Ingestion Service** pulls/streams provider data.
2. **Normalizer** converts raw feed to internal match events.
3. **Live Engine** updates score state in Redis + DB.
4. **Notification Engine** evaluates event rules + user preferences.
5. **Push Service** sends FCM/APNs payloads.
6. **Mobile App** receives push and refreshes live screen via WebSocket/REST.

## 5) Notification Logic (Important)

Each event passes through:
1. Event deduplication (`event_id` uniqueness).
2. User eligibility (follows match/team/tournament).
3. Quiet-hour and frequency checks.
4. Priority mapping:
   - High: wicket/result/super over
   - Medium: milestones/innings break
   - Low: over summaries
5. Delivery + retry with exponential backoff.

## 6) Data Model (Simplified)

- `matches(id, provider_id, series_id, team_a, team_b, format, status, start_time_utc)`
- `innings(id, match_id, number, runs, wickets, overs)`
- `events(id, match_id, type, payload_json, created_at)`
- `users(id, email, timezone, quiet_hours_start, quiet_hours_end)`
- `subscriptions(id, user_id, scope_type, scope_id)` // match/team/tournament
- `notification_log(id, user_id, event_id, status, sent_at)`

## 7) API Design (MVP)

- `GET /matches?status=live&date=YYYY-MM-DD`
- `GET /matches/{id}`
- `GET /matches/{id}/commentary`
- `POST /users/{id}/subscriptions`
- `DELETE /users/{id}/subscriptions/{subId}`
- `PUT /users/{id}/notification-preferences`
- `GET /teams/{id}/matches?range=7d`

WebSocket channels:
- `match:{matchId}:score`
- `match:{matchId}:commentary`
- `user:{userId}:alerts`

## 8) UX Flow

1. Open app → **Live tab** first.
2. Tap match → scorecard + commentary + stats.
3. Tap bell icon → customize alert types.
4. Receive push → deep-link directly into match.

## 9) Non-Functional Requirements

- 99.9% API uptime target during major tournaments.
- P95 API response < 300ms for cached endpoints.
- Event-to-push latency target: < 10s.
- GDPR/DPDP compliant data retention.

## 10) MVP Roadmap (8 Weeks)

- **Week 1–2:** Backend skeleton, DB schema, provider integration.
- **Week 3–4:** Live match APIs + WebSocket updates.
- **Week 5:** Mobile screens (Live/Upcoming/Details).
- **Week 6:** Notification engine + preference settings.
- **Week 7:** QA, load tests on match-day traffic.
- **Week 8:** Beta launch + analytics instrumentation.

## 11) Future Enhancements

- Win probability and AI highlights.
- Widgets/watch app support.
- Multi-language commentary.
- Audio score alerts for accessibility.

---

If you want, the next step can be a **working MVP scaffold** (Flutter app + NestJS backend + WebSocket + push pipeline skeleton).
