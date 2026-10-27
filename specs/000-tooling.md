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
