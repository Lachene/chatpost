PostChat — Viral Website Spec (Ultra-Detailed, UDC Style)
0) One-line concept

PostChat = social feed + friend DMs + moods + status flex. Users post (text / photo / video), react (like/dislike/custom), follow or friend (accept/decline), DM (chat, pics, short videos), and show mood. Viral because: status + scarcity (rare badges, streaks), loops (duets/remixes + share cards), FOMO (invite-only unlocks).

1) User stories (atomic, testable)

As a visitor, I can explore a limited Trending feed before signup (blurred media; tap → signup).

As a new user, I can sign up with phone/email/Google; pick username (unique), name, mood, avatar, banner, bio in ≤60s.

As a user, I can follow anyone; friend requires request → accept/decline.

As a user, I can post text / images / short video (≤60s) / poll.

As a user, I can react (👍 👎 ❤️ 🔥 😂 😡 + mood-react), comment, share.

As a user, I can DM friends (text, image, 15s video, voice note, stickers).

As a user, I can set mood (Happy, Chill, Angry, Focused, …) which colors my UI & reacts for 24h.

As a user, I can change name, avatar, banner any time; username only once every 14 days.

As a user, I can block, report, mute.

As a user, I can see notifications (likes, comments, follows, friend accepts, DM).

As a creator, I can see Insights (views, reach, watch time, CTR, saves, shares).

As a power user, I can duet/remix a post (reply with split layout).

As a paid user (optional), I can equip premium badges/skins; bigger uploads; vanity URL.

2) Information architecture (pages & routes)

/ Landing: hero, sample feed (blurred), CTA Sign up.

/feed Home feed (following + recommendations).

/explore Trending, tags, moods, people to follow.

/post/:id Post detail, comments, duets.

/profile/:username Profile (Posts | Media | About | Friends | Followers).

/messages DM list; /messages/:conversationId chat.

/notifications All, Mentions, Follows, System.

/settings Account, Privacy, Security, Display, Notifications, Data.

/auth/* Login, Signup, Magic link, Password reset.

/admin Moderation queue (internal).

/legal/* Terms, Privacy, Community Guidelines.

3) Visual system (clear, consistent)

Palette: Primary Turquoise-Blue (#14B8A6) + Magenta (#D946EF). Background dark: #0B0F14; light: #F8FAFC. Accents auto-shift by mood:

Happy → warm accent, Chill → cool cyan, Angry → orange/red, Focused → violet.

Typography: Headings Inter 700/800; body Inter 400/500; monospace JetBrains Mono for code snippets.

Spacing scale: 4, 8, 12, 16, 24, 32, 48 px.

Radius: 12 px cards; 999 px pills.

Elevation: 0/2/8/24 shadow tiers; hover elevates +2.

Icon set: Lucide (stroke 1.75); sizes 16/20/24.

Motion: 150–200ms ease-out for UI; 400ms for modal.

4) UX flows (step-by-step, pixel behaviors)
4.1 Onboarding (≤60s)

Phone/email or Google → verify (OTP/magic link).

Pick username (validate live, case-insensitive; reserve on submit), name.

Choose mood (preview theme tint).

Upload avatar (auto-crop to circle) & banner; write bio (≤160 chars).

Suggested follows: 10 accounts (interests: pick 3 tags).

First Action Nudge: “Post your first vibe” (template cards).
Dopamine #1: confetti + starter badge “Day 1”.

4.2 Post creation

Composer: tabs Text / Photo / Video / Poll.

Drag-drop or paste images; video ≤60s, auto-transcode to HLS + poster.

Add caption (≤500), tags (#), visibility: Public / Followers / Friends / Private.

Remixable toggle: on → others can duet.

Schedule (optional): set time; shows “scheduled” chip.

Submit → optimistic insert → progress bar for media → success toast.

4.3 Reactions / comments

Tap react opens 6-slot reaction wheel (👍 👎 ❤️ 🔥 😂 😡) + Mood React (changes with your mood).

One active reaction per user per post; second tap removes; switch replaces.

Comments: flat + inline replies (1 level).

Auto-link hashtags and mentions; long press → copy / report.

4.4 Follow vs Friend

Follow: one-way; posts enter feed.

Friend: request → accept/decline; unlocks DM and Friends-only posts.

UI: “Add Friend” (if following each other, show “Make friends”).

Username change logic: store last_username_change_at; block change if diff < 14 days; show countdown.

4.5 DMs (WhatsApp-smooth)

Threads list: last message, unread badge, typing indicator.

Message states: sent (1 tick), delivered (2 ticks filled), seen (avatar dot).

Attach: image, 15s video, voice note, sticker.

Long-press → reply, forward, delete (you), pin, report.

Typing: emits after 300ms and every 3s while typing.

Read: when window focused and last message in viewport for ≥1s.

4.6 Notifications

Real-time toasts; /notifications groups by type; daily digest email (opt-in).

Batched: multiple likes on same post collapse (“+23 others”).

Push (web) via service worker; respect Do Not Disturb (22:00–08:00 local).

5) Gamification & status

Levels: XP from post, react, comment, streak.

Streaks: Daily post or react keeps streak; missing a day breaks (one Streak Freeze item can preserve).

Badges: Day 1, 7-day Streak, 1k Reactions, “First Duet,” Seasonal (Halloween, New Year).

Seasonal leaderboards: reactions, duets, watch time; top 1% get rare badge (time-limited).

Profile cosmetics: frames, banners, emoji packs, profile themes tied to mood.

6) Safety, privacy, controls

Block: removes both follow & friend; hides content both ways.

Mute: hide from feed, keep relationship.

Report: per post/comment/user/DM with reason; queues to moderation.

Visibility per post (Public/Followers/Friends/Private).

Account: Private (approve followers) or Public.

Minimum age: 13+; COPPA safe.

Download data / Delete account: export JSON/ZIP; 30-day grace (deactivated) before purge.

7) Ranking (feed algorithm)

Signals (scored; decay by time):

Creator relationship (friend > follow > none).

Freshness (half-life 12h; longer for friends).

Engagement velocity (reactions/min), save/share rate, comment quality.

Watch completion for videos (0–1).

Negative feedback (hide/report/👎).

Interest match (user tags vs post tags).

Novelty / remixability (duet count).
Diversification: cap creator frequency, insert “new creators” 1 in 6 cards.
Controls: “Following only” toggle, “I’m not interested” action.

8) Data model (Postgres / Supabase-friendly)
8.1 Core tables
-- Users & profiles
create table auth_users (
  id uuid primary key default gen_random_uuid(),
  email text unique,
  phone text unique,
  created_at timestamptz default now()
);

create table profiles (
  id uuid primary key references auth_users(id) on delete cascade,
  username citext unique not null check (username ~ '^[a-z0-9_]{3,20}$'),
  display_name text not null check (length(display_name) between 1 and 40),
  bio text default '' check (length(bio) <= 160),
  avatar_url text, banner_url text,
  mood text not null default 'Chill',
  is_private boolean default false,
  last_username_change_at timestamptz default now(),
  followers_count int default 0, following_count int default 0,
  friends_count int default 0,
  created_at timestamptz default now(), updated_at timestamptz default now()
);

-- Follows (one-way)
create table follows (
  follower_id uuid references profiles(id) on delete cascade,
  followee_id uuid references profiles(id) on delete cascade,
  created_at timestamptz default now(),
  primary key (follower_id, followee_id),
  check (follower_id <> followee_id)
);

-- Friend requests + friendships
create table friend_requests (
  id bigserial primary key,
  sender_id uuid not null references profiles(id) on delete cascade,
  receiver_id uuid not null references profiles(id) on delete cascade,
  status text not null check (status in ('pending','accepted','declined','canceled')),
  created_at timestamptz default now(),
  unique (sender_id, receiver_id),
  check (sender_id <> receiver_id)
);

create table friendships (
  user_a uuid not null references profiles(id) on delete cascade,
  user_b uuid not null references profiles(id) on delete cascade,
  created_at timestamptz default now(),
  primary key (user_a, user_b),
  check (user_a < user_b) -- canonical pair
);

-- Posts
create table posts (
  id bigserial primary key,
  author_id uuid not null references profiles(id) on delete cascade,
  type text not null check (type in ('text','image','video','poll')),
  caption text,
  visibility text not null default 'public' check (visibility in ('public','followers','friends','private')),
  remixable boolean default true,
  parent_post_id bigint references posts(id) on delete set null, -- for duets/remix
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

-- Media (images/videos)
create table media (
  id bigserial primary key,
  post_id bigint references posts(id) on delete cascade,
  kind text check (kind in ('image','video')),
  url text not null,
  width int, height int, duration_seconds numeric,
  blurhash text,
  created_at timestamptz default now()
);

-- Reactions
create table reactions (
  post_id bigint references posts(id) on delete cascade,
  user_id uuid references profiles(id) on delete cascade,
  kind text not null check (kind in ('like','dislike','love','fire','laugh','angry','mood')),
  created_at timestamptz default now(),
  primary key (post_id, user_id)
);

-- Comments
create table comments (
  id bigserial primary key,
  post_id bigint references posts(id) on delete cascade,
  author_id uuid references profiles(id) on delete cascade,
  parent_id bigint references comments(id) on delete cascade,
  body text not null check (length(body) <= 1000),
  created_at timestamptz default now()
);

-- Messages
create table conversations (
  id bigserial primary key,
  a uuid references profiles(id) on delete cascade,
  b uuid references profiles(id) on delete cascade,
  created_at timestamptz default now(),
  unique(a,b) -- enforce single thread per pair (ensure a<b at insertion)
);

create table messages (
  id bigserial primary key,
  conversation_id bigint references conversations(id) on delete cascade,
  sender_id uuid references profiles(id) on delete cascade,
  body text,
  kind text not null check (kind in ('text','image','video','voice','sticker')),
  media_url text,
  status text not null default 'sent' check (status in ('sent','delivered','seen')),
  created_at timestamptz default now()
);

-- Notifications
create table notifications (
  id bigserial primary key,
  user_id uuid references profiles(id) on delete cascade,
  type text not null, -- like, comment, follow, friend_accept, mention, system
  actor_id uuid references profiles(id) on delete set null,
  post_id bigint references posts(id) on delete set null,
  comment_id bigint references comments(id) on delete set null,
  read boolean default false,
  created_at timestamptz default now()
);

-- Moderation & safety
create table blocks (
  blocker_id uuid references profiles(id) on delete cascade,
  blocked_id uuid references profiles(id) on delete cascade,
  created_at timestamptz default now(),
  primary key (blocker_id, blocked_id)
);

create table reports (
  id bigserial primary key,
  reporter_id uuid references profiles(id),
  target_kind text check (target_kind in ('post','comment','user','message')),
  target_id bigint,
  reason text,
  extra jsonb,
  status text default 'open' check (status in ('open','reviewing','actioned','dismissed')),
  created_at timestamptz default now()
);

-- Analytics (events)
create table events (
  id bigserial primary key,
  user_id uuid,
  name text not null, -- VIEW_POST, LIKE, FOLLOW, START_DUET, WATCH_COMPLETE, DM_SEND
  props jsonb default '{}',
  created_at timestamptz default now()
);

Note: Foreign key names will auto-generate, but if you want exact names for Supabase hints, add constraint friend_requests_sender_id_fkey foreign key (sender_id) references profiles(id) etc.

8.2 Indexing (performance)

create index on posts (author_id, created_at desc);

create index on posts (created_at desc);

create index on reactions (post_id);

create index on comments (post_id, created_at);

create unique index on conversations (least(a,b), greatest(a,b));

create index on notifications (user_id, read, created_at desc);

Full-text: tsvector on posts.caption and comments.body.

8.3 Counters (consistency)

Use triggers or background jobs to maintain followers_count, friends_count, reaction totals and comment counts. Prefer transactional updates on write to avoid N+1 reads.

9) Real-time architecture

WebSockets (Supabase Realtime or Pusher/Ably): channels

feed_global, post:{id}, user:{id}:notifications, dm:{conversationId}.

Events:

New post → push to followers & friends channels.

New reaction/comment → push to post channel.

Friend request accepted → notify both.

DM send → emit message:new; server updates status delivered on ack; seen when client emits read.

10) API (clean, versioned)
10.1 REST (example; GraphQL also fine)

POST /v1/auth/signup {email|phone, code} → {user}

GET /v1/feed?cursor= → paginated mixed feed

POST /v1/posts {type, caption, media[], visibility, remixable, parent_post_id?}

GET /v1/posts/:id

POST /v1/posts/:id/react {kind}

POST /v1/posts/:id/comment {body, parent_id?}

POST /v1/follow/:userId / DELETE /v1/follow/:userId

POST /v1/friend-requests {receiver_id}; POST /v1/friend-requests/:id/accept

GET /v1/profile/:username → profile + counts

PATCH /v1/profile {display_name, bio, avatar_url, banner_url, mood}

PATCH /v1/profile/username {new_username} (server checks 14-day rule)

GET /v1/messages/:conversationId (cursor, oldest-first)

POST /v1/messages/:conversationId {kind, body?, media_url?}

GET /v1/notifications (read filter, cursor) / PATCH /v1/notifications/read

Error codes: 400_INVALID_INPUT, 401_UNAUTH, 403_FORBIDDEN, 404_NOT_FOUND, 409_CONFLICT (username, duplicate follow), 429_RATE_LIMIT, 500_INTERNAL.

11) Rate limits (per IP + per user)

Signup OTP: 5/hour.

Post create: 15/day.

DM sends: 60/minute burst, 600/hour.

Reactions: 30/minute, debounced 500ms.

Comments: 10/minute.

Username changes: enforced 14-day cooldown.

12) Storage & media

Object storage (S3/Backblaze) buckets:

avatars/, banners/, posts/{postId}/, messages/{conversationId}/.

Image transforms: 256, 512, 1024; WebP; generate blurhash.

Video: ingest → queue → FFmpeg HLS (360p/720p), poster frame, duration.

CDN cache rules: immutable URLs w/ content hash; signed URLs for private visibility.

13) Security

Auth: Supabase Auth or Firebase Auth (JWT).

Passwords: argon2id (if self-auth).

RLS policies (Supabase):

Users read public profiles; private profiles visible to approved followers & friends.

Posts visibility enforced in select policy.

DMs: only conversation participants can select/insert.

CSRF for cookie flows; CSP default-src 'self'; strict transport security.

Abuse: IP throttling, device fingerprint (soft), disposable email denylist (optional).

Backups: nightly Postgres; 7 daily, 4 weekly, 6 monthly rotations.

14) Moderation

Automated: image/video NSFW classifier; toxic text filter; URL safety check.

Human: /admin queue: report triage, action (delete, shadowban, warn).

Shadowban option: user can post but reach is 0; review in 7 days.

Appeals: in-app form; SLA 72h.

15) Analytics (event taxonomy)

Identity: USER_SIGNUP, COMPLETE_ONBOARDING, SET_MOOD.

Content: CREATE_POST, VIEW_POST, WATCH_25/50/95, LIKE/UNLIKE, COMMENT, SHARE_COPYLINK, START_DUET, PUBLISH_DUET, SAVE_POST.

Social: FOLLOW, UNFOLLOW, FRIEND_REQUEST_SEND/ACCEPT, MESSAGE_SEND/SEEN.

Safety: REPORT_SUBMIT, BLOCK, MUTE.

Monetization (optional): SUBSCRIBE_START, SUBSCRIBE_RENEW, PURCHASE_COSMETIC.
Dimensions: user_id, post_id, creator_id, device, country, mood, session_id.
KPIs: DAU/WAU/MAU, Day-1/7/30 retention, median session length, creation rate, share rate, invite K-factor, streak adherence, feed satisfaction (hide/not interested rate).

16) Virality engines

Share Cards: server-rendered OG images for posts & profiles (big, clean, on-brand).

Duet/Remix: one-tap reply with side-by-side layout; original credited; remix chain visible.

Invite Locks: some cosmetics & “Explore Pro” unlock with 3 invited friends.

Streak Flex: profile ring color changes with streak length (rare colors for 30/60/100).

Copy-Link Boost: copying a link adds a tiny boost weight to ranking.

Seasonals: limited-time tags, challenges, leaderboards (Top “Chill” creators week).

17) Monetization (optional, cosmetics-first)

Free: all core social features.

Plus ($3–5/mo): golden badge, animated profile frame, extra themes, longer videos (120s), scheduled posts, advanced Insights.

One-time: seasonal skin packs, rare frames, sticker packs.

No ads by default.

18) Accessibility & intl

WCAG 2.1 AA: color contrast ≥ 4.5:1; keyboard navigable; focus rings.

Captions for video; alt text required for images.

i18n: en → az → tr. Date/time localized; RTL support ready.

19) Performance & SLOs

TTI < 3s on 3G; LCP < 2.5s; CLS < 0.1.

API p95 < 300ms; media via CDN.

Outage comms: /status page; incident posts.

20) Tech stack (recommended)

Frontend: Next.js (App Router) + React + TypeScript + Tailwind; Zustand/Redux for state.

Backend: Supabase (Postgres + Realtime + Storage) or Node/NestJS + Postgres + Redis.

Realtime: Supabase Realtime or Pusher/Ably.

Jobs: BullMQ / Cloud Tasks for video transcode, digests, counters.

Search: Postgres FTS; optional OpenSearch for scale.

CDN: Cloudflare; Image Resizing; HLS on CDN.

Payments: Stripe (if monetizing).

Infra: Vercel (web), Fly.io/Render (API), R2/S3 (media).

21) Component inventory (frontend)

PostCard (media, caption, reacts, share, duet).

ComposerModal (tabs, uploader, visibility).

ReactionBar (wheel, counts).

CommentList + CommentItem.

ProfileHeader (avatar, banner, stats, mood ring).

FollowButton, FriendButton.

MoodPicker (animated icons).

DMThread, MessageBubble, TypingDots.

NotificationItem (grouping logic).

InsightsPanel (charts).

InviteDialog.

ReportDialog, BlockConfirm.

SettingsForms (account, privacy, security).

FeatureFlagGate (A/B).

22) Edge cases (don’t forget)

Username normalization lowercase; reserve on selection; 14-day cooldown enforcement.

Blocking cancels pending friend requests and unfollows both ways.

Private account: follow requests appear in a tab; until approved, follower can’t see.

Deleted post: reactions/comments cascade delete; notifications soft-hide.

DM safety: if either user blocks, auto-archive conversation.

Video upload fails → keep draft; retry queue.

Rate limit strikes show inline (“Try again in 17s”).

Time skew: show “~just now” if server <-> client off by <60s.

23) QA checklist (pre-launch)

Signup OTP retry & lockout.

Username rule tests (regex, uniqueness, cooldown).

Media: formats (jpg/png/webp/mp4), corrupt file handling.

Feed: pagination (cursor), dedupe.

DMs: typing/seen accuracy with tab hidden.

Notifications: grouping & badge count correctness.

RLS tests: unauthorized selects fail.

Accessibility: keyboard paths; alt text enforcement.

24) Launch plan (30 days)

Week 1: closed alpha (100 users). Measure: creation rate, 7-day retention.

Week 2: add Duet/Remix + Share Cards + Invite Locks.

Week 3: seasonal challenge + streak cosmetics.

Week 4: open beta with waitlist codes; creator seeding.

25) Copy blocks (ship-ready)

Empty feed: “Your feed is lonely. Follow 5 people or post your first vibe.”

First post helper: “Try a 10-second video or 3 photos—remixable on.”

Username change banner: “Next change in 13 days.”

Streak toast: “🔥 Day 7! You unlocked ‘Amber Ring’.”

Invite lock: “Unlock ‘Chill Neon’ theme by inviting 3 friends.”

26) Minimal MVP checklist (what to build first)

Auth + Profiles (username cooldown).

Follow/Friend + DMs (text + images).

Posts (text/image/video) + reactions + comments.

Feed (following + basic trending).

Mood system + mood-react.

Notifications (in-app).

Basic moderation (report/block).

Image/video storage + CDN.

A/B flags for Duet + Invite Locks (can ship in week 2).
