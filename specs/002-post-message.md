
# 002 - Post a message

**Goal:** a form to post a message, with validation, rate limiting and cache
invalidation.

**Behavior**
- A form with `author` and `body` (+ a hidden honeypot field) on `/`.
- Server Action. Validates with zod: `author` 2-24 and `body` 1-140 characters,
  both trimmed. If the honeypot has content, respond as success without saving.
- Rate limit with `@upstash/ratelimit` (sliding window): 3 submissions per 60 s
  per client, using a hash of the IP (`x-forwarded-for`), never the raw IP.
  If Redis does not respond: allow the submission (fail-open) and log the error.
- Inserts into `public.messages` using the secret/service-role key, SERVER-ONLY.
- After inserting: delete `feed:v1` and call `revalidatePath('/')`.
- Typed errors back to the form (invalid field, rate limited, generic error).
  Do not expose internal messages or keys.

**Acceptance**
- A valid message appears in the feed after submitting it.
- Invalid input shows per-field errors and does not insert.
- The 4th submission within a minute returns the rate-limit error.
- Tests: zod schema, action with Supabase/Redis mocked (success, invalid,
  rate limited, honeypot).

**Out of scope:** user accounts, moderation, editing/deleting messages.

