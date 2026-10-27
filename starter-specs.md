# Bootstrap pack for the "launch-wall" project

This file holds every SDD artifact the repo needs before any code exists. In the
repo they live as separate files. Each file starts at a `<!-- FILE: path -->` marker
and ends right before the next marker (or the end of this file); the marker line
itself is not copied:

- `specs/README.md` (index and execution order)
- `specs/000-tooling.md`, `specs/004-design.md`, `specs/001-feed.md`,
  `specs/002-post-message.md`, `specs/003-health.md`
- `design/Design.md`, `design/magicpath/README.md`
- `docs/superpowers/specs/.gitkeep`, `docs/superpowers/plans/.gitkeep` (empty)

Codex splits this file mechanically (no content changes) and then deletes it.

Variable names: after installing the Vercel integrations (Supabase and Upstash),
check which names they actually inject and adjust the code and `.env.example`
to those names.

<!-- FILE: specs/README.md -->

# Specs index

`specs/NNN-*.md` define WHAT (requirements and acceptance criteria) and are owned by
the human. Superpowers writes the HOW: design docs in `docs/superpowers/specs/` and
plans in `docs/superpowers/plans/`. One thread and one branch per spec.

Execution order (004 runs second because the UI specs depend on the prototype):

| Order | Spec | Status |
|---|---|---|
| 1 | 000-tooling | todo |
| 2 | 004-design (MagicPath prototype + theme) | todo |
| 3 | 001-feed | todo |
| 4 | 002-post-message | todo |
| 5 | 003-health | todo |

Update the Status column (todo / in progress / done) in the PR that changes it.

<!-- FILE: specs/000-tooling.md -->

# 000 - Tooling and quality

**Goal:** a Next.js project with automated quality, so that style and rules are
enforced by tools and not by the agent's memory.

**Scope**
- Next.js app (App Router, TypeScript, Tailwind, ESLint) in the current folder,
  with shadcn/ui initialized. The folder is not empty (AGENTS.md, specs/, docs/,
  design/), so `create-next-app` may refuse to run in it: scaffold in a temporary
  folder and move the files in, merging `.gitignore` and never overwriting AGENTS.md,
  README.md, specs/, docs/ or design/.
- `package.json` scripts:
  - `lint`: `eslint .`
  - `format`: `prettier --write .`
  - `format:check`: `prettier --check .`
  - `typecheck`: `tsc --noEmit`
  - `test`: `vitest run --passWithNoTests`
  - `check`: lint + format:check + typecheck + test
- ESLint: flat config (`eslint.config.mjs`) based on the one create-next-app
  generates; add `eslint-config-prettier` last to avoid conflicts. Ignore
  `.next`, `node_modules`, `coverage`, `design/magicpath`.
- Prettier: `.prettierrc.json` with semi, singleQuote, trailingComma "all",
  printWidth 100. Add `.prettierignore` (including `design/magicpath` and
  `docs/superpowers`) and `.editorconfig`. Run `npm run format` once so the existing
  Markdown passes `format:check`.
- `tsconfig.json` and Vitest exclude `design/magicpath` (reference code, not shipped).
- VS Code (committed): `.vscode/settings.json` with format on save, Prettier as
  the default formatter and `source.fixAll.eslint` on save;
  `.vscode/extensions.json` recommending ESLint, Prettier, SonarQube for IDE
  (`SonarSource.sonarlint-vscode`) and Tailwind CSS IntelliSense.
- Git hooks: husky + lint-staged. The pre-commit hook runs `eslint --fix` and
  `prettier --write` on staged files.
- Vitest installed and configured.
- `.env.example` (names only) and `.env*` in `.gitignore` (except the example).
- `sonar-project.properties` with sources `app,components,lib`, exclusions for
  `node_modules` and `.next`; projectKey/organization left as TODOs to fill in by hand.

**Acceptance**
- `npm run check` and `npm run build` pass from a clean state.
- Files under `design/magicpath/` do not affect lint, format, typecheck or tests.
- A useless `console.log` or a badly formatted file makes `check`/commit fail.

**Out of scope:** CI, Supabase, Redis, product UI.

<!-- FILE: specs/004-design.md -->

# 004 - Design (MagicPath prototype)

**Goal:** an interactive prototype of the single launch-wall page in MagicPath, turned
into project design tokens, plus an honest evaluation of the tool.

**Human part (outside Codex)**
- In a personal MagicPath account (Free plan, no card), design one page with: header,
  post form (author, message, character counter), feed list (author, text, relative
  time), and the states: empty feed, field validation errors, rate-limited message,
  generic error, success. Demo content only.
- Export the code (React + TypeScript + Tailwind) and copy it into `design/magicpath/`.
- Write down the credits used and the time it took.

**Codex part**
- Read `design/magicpath/` as reference only (never import from it).
- Fill `design/Design.md`: colors, typography, radius, spacing, tone, component list,
  and a mapping from each prototype element to a shadcn/ui primitive.
- Apply the tokens to the shadcn/Tailwind theme (CSS variables in the global stylesheet)
  and add the shadcn components the mapping needs. No product UI yet (that is 001/002).
- Create `design/EVALUATION.md` with: credits used, time, how many lint/type errors the
  exported code would raise under the project rules, how much of it maps cleanly to
  shadcn/ui, and a verdict (keep / drop MagicPath for the next project) with reasons.

**Acceptance**
- `design/Design.md` and `design/EVALUATION.md` are complete; theme tokens applied.
- `npm run check` and `npm run build` pass; `design/magicpath/` stays excluded.

**Out of scope:** implementing the feed or the form (001, 002); Figma.

<!-- FILE: specs/001-feed.md -->

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

<!-- FILE: specs/002-post-message.md -->

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

<!-- FILE: specs/003-health.md -->

# 003 - Health check

**Goal:** an endpoint to quickly verify that Vercel, Supabase and Redis are
connected (useful on every preview).

**Behavior**
- `GET /api/health` returns JSON `{ ok, supabase: "up"|"down", redis: "up"|"down" }`.
- Supabase: a minimal query (for example a `select` of 1 row with limit 1).
  Redis: `PING`.
- 200 if everything is "up"; 503 if anything is "down". Header `Cache-Control: no-store`.
- Do not include variable names, URLs, keys or error details in the response.

**Acceptance**
- On the Vercel preview it returns 200 with both services "up".
- Test: the response becomes 503 when a service fails (dependencies mocked).

**Out of scope:** metrics, authenticating the endpoint.

<!-- FILE: design/Design.md -->

# Design notes (filled in spec 004)

Source: MagicPath prototype exported to `design/magicpath/` (reference only).

## Tokens
- Colors: TODO
- Typography: TODO
- Radius and spacing: TODO

## Tone
TODO

## Element to component mapping
| Prototype element | shadcn/ui primitive | Notes |
|---|---|---|
| TODO | TODO | TODO |

<!-- FILE: design/magicpath/README.md -->

# MagicPath export (reference only)

Code exported from the MagicPath prototype. It is NOT part of the app:
- never import from this folder;
- it is excluded from lint, format, typecheck, tests and Sonar;
- screens are rebuilt in `app/` and `components/` with shadcn/ui and the project tokens.

Replace the contents of this folder when the prototype changes, in its own commit.
