# AGENTS.md - launch-wall

Project instructions. General rules (quality, git, secrets, safety) live in the global
`~/.codex/AGENTS.md`; this file only adds what is specific to this project.
If a rule here conflicts with a spec in `specs/`, stop and ask.

## Stack

- Next.js (App Router, TypeScript strict), React Server Components by default
- Tailwind CSS + shadcn/ui (components live in `components/ui`)
- Supabase (Postgres, Auth, Storage) with Row Level Security
- Redis (Upstash, via Vercel Marketplace) for cache and rate limiting
- Vercel (hosting; preview deployment per PR, production from `main`)
- MagicPath for the interactive prototype (design source; see "Design")

## Workflow: Superpowers (spec-driven)

This project uses the Superpowers plugin skills instead of a hand-made SDD loop.
Load `using-superpowers` first; it routes each task to the right skill. If a skill is
not installed or not available, tell the human instead of improvising the workflow.

Division of labor (keep it this way to avoid duplicate documents):

- `specs/NNN-*.md` = WHAT: the human-owned requirements and acceptance criteria.
  Never rewrite them without asking. Update them in the same PR when behavior changes.
- `docs/superpowers/specs/` = HOW (design) and `docs/superpowers/plans/` = the task plan,
  both written by the skills. Do not move them or create parallel plan files.

Per task, one thread per `specs/` file:

1. `brainstorming`: start from the existing spec; do not re-ask what it already answers.
   Ask only about real gaps. Stop at the design approval checkpoint and wait for the human.
2. `using-git-worktrees`: work on a branch in an isolated worktree (never on `main`).
3. `writing-plans`: small tasks, each with files, a failing test or verification, the
   command to run and its expected output.
4. Execute with `executing-plans` (inline, default: cheaper). Use
   `subagent-driven-development` only when the human asks: subagents use noticeably more
   tokens and inherit this session's sandbox and approval settings.
5. `test-driven-development` for logic (schemas, cache helpers, actions); a failing test
   first, then the code. UI-only changes are verified against the prototype instead.
6. `requesting-code-review` between tasks; fix blocking findings before continuing.
7. `verification-before-completion` then `finishing-a-development-branch`: run the
   Definition of done and open a PR. Do not merge or discard a branch without approval.

Rules that override a skill when they conflict (user instructions take priority):

- Ask before adding dependencies, changing CI/CD, auth/RLS or billing, even if a plan
  lists it. Tell the human which plan step needs it.
- Never skip the human approval checkpoints, and never bypass hooks with `--no-verify`.
- Keep reports short. Do not paste whole plans or specs into the chat; reference paths.
- Full GitHub Spec Kit (`.specify/`) is not used in this project.

## Commands

- Install: `npm ci`
- Dev server: `npm run dev`
- All quality checks: `npm run check` (lint + format:check + typecheck + test)
- Auto-fix style: `npm run format` and `npm run lint -- --fix`
- Build: `npm run build`

## Design (MagicPath prototype)

- The human designs the prototype in MagicPath; it is the visual reference, not the
  source of truth for behavior (that is `specs/`). Mention it during `brainstorming`.
- Exported code (React + TypeScript + Tailwind) is reference material. Put it in
  `design/magicpath/` (excluded from lint, format and Sonar) or read it from the connected
  MagicPath project. Never ship it as-is.
- When implementing a screen: rebuild it in `app/` and `components/` using shadcn/ui
  primitives, the project's Tailwind tokens and the conventions below. Match layout,
  spacing, hierarchy and states of the prototype; do not copy its styling hacks.
- Do not invent UI that is not in the prototype or the spec. If they disagree, ask.
- Replace prototype placeholder data with real data flows from the spec. No fake data
  committed outside tests.
- Keep Design notes (tokens, fonts, colors, tone) in `design/Design.md` if it exists.
- Use only demo content in the prototype: no work designs, brand assets or company data.

## Conventions

- No `any` unless justified in a comment.
- Prefer Server Components and Server Actions. Add `"use client"` only when needed
  (state, effects, browser APIs).
- Validate all external input (forms, route handlers, Server Actions) on the server with
  a schema (zod).
- Use Tailwind utilities and shadcn/ui primitives. No other UI library without approval.
  Add components with the shadcn CLI/MCP.
- Check the Next.js version in `package.json`; use the Next.js DevTools MCP or official
  docs for version-specific APIs instead of relying on memory.
- Tests: unit tests for logic (schemas, cache helpers, actions with mocked dependencies).
  No end-to-end tests unless a spec asks for them.
- SonarQube: new code must pass the quality gate (no new bugs, vulnerabilities or
  security hotspots; low cognitive complexity; no duplicated blocks). If the SonarQube MCP
  server is available, analyze the changed files before finishing.

## Supabase rules

- Every table in `public` has RLS enabled and explicit policies. Views that expose data
  must respect RLS (`security_invoker`).
- Schema changes go through migrations in `supabase/migrations/`
  (`supabase migration new <name>`). Never edit a migration that was already applied or
  merged; create a new one.
- The service-role / secret key is server-only: never in client code, never in
  `NEXT_PUBLIC_*` variables.
- No destructive SQL without human approval.

## Cache rules (Redis)

- The cache is an optimization, never a dependency: if Redis fails, fall through to the
  source of truth and keep the page working.
- Every key has a version prefix (`feed:v1`) and a TTL. Invalidate on writes.
- Never cache per-user or secret data under a shared key.

## Environment

- Secrets live in Vercel environment variables and git-ignored `.env.local`. This
  repository is public: never commit secrets, tokens or `.env*` files.
- Document every new variable in `.env.example` (names only, no values).

## Deployment

- Vercel deploys previews from PRs and production from `main`. Do not add a deploy step
  to GitHub Actions or run `vercel --prod`.
- GitHub Actions is CI only. The required status check is the `ci` job.

## Definition of done

- Spec updated; acceptance criteria met; the screen matches the prototype.
- `npm run check` and `npm run build` green; Sonar quality gate green on the PR.
- Preview deployment works for the changed flow.
- No new secrets, no disabled RLS, no unrelated changes in the diff.
