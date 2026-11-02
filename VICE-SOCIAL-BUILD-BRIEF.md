# VICE SOCIAL: Build Brief for Claude Code

A GTA VI community platform. Build the real backend and rebuild the prototype UI on it.
Reference prototype: https://claude.ai/artifact/23F7NFrnxiuSfwi7Kd3tEa (single HTML file, browser-only storage). Port its behavior, not its storage.

## Hard rules
1. No AI anywhere: no AI recommendations, moderation, translation, chatbots or generated content.
2. No fake data, fake users or fabricated statistics. Empty states instead.
3. Never invent GTA VI information. Every game-fact record has a verification status; official status is set only by moderators with a source link.
4. Every setting is enforced by the backend (row-level security and API checks), never by hiding buttons.
5. No secrets in frontend code. Document every credential in `.env.example`.
6. A feature is "done" only when it has UI, API, DB persistence, permission checks, validation, loading/empty/error states, and a test.
7. Mark anything blocked on external credentials as "needs configuration" and never pretend it works.

## Stack
- Next.js (App Router) + TypeScript, Tailwind. PWA manifest and service worker.
- Supabase: Postgres with row-level security, Auth (email, password reset, TOTP 2FA), Realtime (chat, presence, typing), Storage.
- Video: upload to object storage, background transcoding with FFmpeg worker (HLS renditions, thumbnails). Enforce 6 s to 60 min, size and format limits server-side.
- Voice rooms and watch parties: LiveKit (WebRTC) or equivalent. Needs credentials.
- Twitch: official Helix API, app token on server, OAuth for account linking. Only the GTA VI category is ever queried or displayed. Respect embed rules and rate limits.
- Search: Postgres full-text search with GIN indexes and trigram.
- Jobs: queue worker for transcoding, notifications, export generation, premiere/event reminders, story expiry.
- Rate limiting and input validation (zod) on every route. SSRF protection for RSS fetching. Upload validation by magic bytes.

## Design
Dark charcoal/black base, neon pink `#ff2e93`, purple `#7c3cff`, orange `#ff8a2b`. Unbounded (display) + Figtree (body). Light theme optional. Responsive, keyboard accessible, reduced-motion respected, text scaling, high contrast.
Navigation (six destinations): Home, Explore, Media, Leonida, Communities, Profile. Prominent Create button.

## What the prototype already defines (keep the same rules)
- Feed sorts: newest, trending (engagement / age^1.5), top, most discussed, following.
- Discussion categories (18): General GTA VI, Vice City, Leonida, Jason and Lucia, Story and missions, Characters, Vehicles, Weapons, Gameplay mechanics, Online and multiplayer, Easter eggs, Mysteries, News and updates, Theories and speculation, Technical support, Fan creations, Photography, Roleplay.
- Comment sorts: best, newest, controversial. Author can mark one useful answer. Polls: 2 to 6 options, optional close date, one vote per account, results hidden until you vote or the poll closes.
- Database entry types: vehicles, characters, locations, easter eggs, wildlife, weapons, missions. Status: Unverified theory (flag) / Unverified / Supported by community (3+ confirms and more than 2x challenges) / Disputed (3+ challenges and challenges >= confirms). Add Officially confirmed, Debunked (moderator only, source required). Authors cannot verify their own entries.
- Spoiler flag on posts, threads and entries; per-user setting hide/show; author always sees own.
- Looking for players: PS5 or Xbox listed separately (never assume cross-platform), language, style, voice, group size, optional session time stored in UTC with organizer timezone.
- Events: UTC storage, weekly recurrence done with a proper RRULE library in the organizer's timezone (handles DST), RSVP, max participants, .ics export, reminders.
- Notifications: follow, comment/reply, like, useful answer, group join, event, mention; per-type toggles and pause-all; dedupe unread; blocked users never notify.
- Achievements (computed from real activity): first post, first discussion, 10 comments, helpful answer, supported entry, 10 likes from others, 5 image posts, group of 3+. Points: 1 per post/reply, 2 per discussion, 1 per like from others, 5 per useful answer, 5 per supported entry. Own likes never count.
- Private journal and manual completion tracker (no pre-filled lists; no game-save sync).
- Edit history with restore (last 20 versions), report with reason, account deletion that removes all owned data, data export.

## Phases (build in order; ship each phase working)
1. **Core**: auth (email verify, reset, 2FA, sessions list, logout all), profiles (custom URL /@name, banner, avatar, themes, tabs), feed (text, images, galleries, polls, links), reactions (like, love, funny, impressive, interesting, helpful, surprising, controversial), comments, follow, friends, block, mute, close friends, privacy and notification settings, search, rate limits.
2. **Media**: image galleries, video upload + transcoding + playback (short vertical feed, standard player with quality/speed/chapters/timestamps), playlists, creator channels, subscriptions, analytics (only measurable metrics), premieres, watch history, podcasts.
3. **Community**: discussions, Q&A with reputation, communities (public/private/invite, rules acceptance, roles, mods), channels with realtime chat (reactions, replies, pins, attachments, typing, presence), DMs and groups, voice rooms, notifications center.
4. **Live**: Twitch (GTA VI only): live list, filters, embeds, follow, go-live notifications, streamer dashboard.
5. **Leonida**: interactive map (user markers, community vs verified labels, ratings, collections, routes), encyclopedia, wiki with revisions, vehicles, characters (spoiler controls), discoveries timeline, wildlife and weather journals, guides, mission pages, investigation boards, relationship explorer.
6. **Social expansion**: crews, LFG, events and calendars, competitions, awards, trivia, guess-the-location/vehicle, brackets, debates, predictions (no gambling), leaderboards, hall of fame, magazines, stories (24 h expiry), collaboration hub, referral (cosmetic only).
7. **Trust and finish**: admin dashboard, reports queue, appeals, copyright reporting, moderation logs, transparency report, status page (real monitoring only), public API with keys/scopes/rate limits, embeds, human translation contributions, PWA offline, accessibility audit, load tests, deployment docs.

## Core tables (start here)
users, profiles, sessions, follows, friendships, blocks, mutes, posts, post_media, reactions, comments, hashtags, notifications, notification_prefs, privacy_settings, reports, moderation_actions, discussions, discussion_comments, votes, polls, poll_options, poll_votes, entries (+ entry_votes, entry_sources), lfg_requests, lfg_members, events, event_rsvps, journal_entries, tracker_items, achievements_earned, edit_versions, media_assets, videos, video_renditions, playlists, channels, messages, conversations, twitch_links.
Every table: RLS policies written first; tests for blocked users, private accounts and direct-URL access.

## Credentials needed (document, never commit)
Supabase URL + keys, storage bucket, SMTP/email provider, Twitch client ID + secret, LiveKit (or other WebRTC) keys, web push VAPID keys, optional translation-service key (human translation preferred).

## Definition of done per phase
Typecheck, lint, unit + API permission tests, mobile and desktop screenshots reviewed, no dead buttons, README updated with setup steps and what remains.
