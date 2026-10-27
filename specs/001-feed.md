# 001 - Feed

**Goal:** a `/` page that shows the latest 20 wall messages, cached in Redis.

**Data model (migration `supabase/migrations/<timestamp>_create_messages.sql`)**

```sql
create table public.messages (
  id uuid primary key default gen_random_uuid(),
  author text not null check (char_length(author) between 2 and 24),
  body text not null check (char_length(body) between 1 and 140),
  created_at timestamptz not null default now()
);
create index messages_created_at_idx on public.messages (created_at desc);

alter table public.messages enable row level security;

-- Public read. There is NO insert policy: writes go through the server.
create policy "messages_select_public"
  on public.messages for select
  to anon, authenticated
  using (true);
```

**Behavior**

- Server Component. Reads `feed:v1` from Redis; on a hit, uses it.
- On a miss, queries Supabase with the public/anon key (RLS applies), orders by
  `created_at desc`, limit 20, and stores the result in Redis with a 60-second TTL.
- If Redis fails (timeout/error), the page keeps working by reading from Supabase.
- Friendly empty state. Each item shows author, text and relative time in
  English (`Intl.RelativeTimeFormat`).
- UI with Tailwind + shadcn (Card). A single page, no authentication.

**Acceptance**

- With test data, the page lists messages from newest to oldest.
- A second load within 60 s does not query Supabase (verifiable with a temporary
  log or a test of the cache helper).
- Turning off/invalidating Redis does not break the page.
- Tests: cache helper (hit, miss, Redis failure).

**Out of scope:** pagination, search, authentication.
