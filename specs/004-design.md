
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

