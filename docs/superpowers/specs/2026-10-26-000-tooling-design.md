# 000 tooling and quality: design

## Intent

Establish the Next.js foundation for launch-wall and make its style, types, tests, and build reproducible through repository commands. Contributors should get the same quality results locally and at commit time. The human-owned requirements are in [`specs/000-tooling.md`](../../../specs/000-tooling.md).

## Goals and boundaries

- Create the App Router, strict TypeScript, Tailwind, ESLint, and shadcn/ui foundation in the current repository.
- Provide the exact scripts and configuration named in the spec, with `npm run check` and `npm run build` passing from a clean checkout.
- Make staged formatting and lint corrections automatic through Husky and lint-staged, while rejecting remaining lint errors. Treat `console.log` as an ESLint error so the acceptance example fails `check` and a commit.
- Exclude `design/magicpath/` from lint, formatting, type checking, and tests. Keep it as reference material.
- Do not add CI, Supabase, Redis, or product screens. The starter page may remain until its own spec replaces it.

An outcome that passes checks only by suppressing rules, ignores prototype exports inconsistently, overwrites existing project documents, or commits environment values fails this design.

## Approaches

1. **Selected: scaffold in a temporary folder and merge.** Generate a current Next.js App Router project with TypeScript, Tailwind, and ESLint, then copy the app and configuration files into this repository. Merge generated ignore rules with the project's environment rules. This preserves framework defaults while protecting the existing docs and specs. The merge needs explicit file review.
2. **Assemble the configuration by hand.** This gives exact control over every file but risks missing or diverging from the current generator defaults. It offers no benefit for this empty app repository.

## Structure and behavior

1. Generate the app outside the repository, then merge only the needed generated files. Keep `AGENTS.md`, `README.md`, `specs/`, `docs/`, and `design/` intact. Do not bring generated Git metadata into this repository.
2. Initialize shadcn/ui using the generated Tailwind setup. Keep its generated primitives under `components/ui` and use the project's Tailwind tokens. Install only packages needed for the dependencies and tooling explicitly named in the spec; ask before any extra dependency.
3. Add the spec's npm scripts. `check` runs lint, `format:check`, typecheck, and test in sequence. Vitest is configured with `--passWithNoTests`, leaving logic tests for later specs.
4. Keep the generated flat ESLint configuration as a base. Put `eslint-config-prettier` last, ignore generated output and `design/magicpath/`, and make `no-console` an error. Configure Prettier, EditorConfig, and VS Code settings exactly as specified. Run the configured fixers on included files rather than hand-formatting them.
5. Configure strict TypeScript and Vitest exclusions for `design/magicpath/`. Configure Husky and lint-staged so staged source files receive `eslint --fix` and staged supported files receive `prettier --write`; unresolved lint errors stop the commit. Do not bypass hooks.
6. Add an empty, tracked `.env.example` with variable names only when variables are actually required. Git ignores `.env*` except this example. Add `sonar-project.properties` with the required source and exclusion settings and explicit TODO placeholders for project identity.

## Verification

- Run `npm run format` and `npm run lint -- --fix`, then `npm run check` and `npm run build` from the project root.
- Confirm a temporary `console.log` and a temporary badly formatted file cause the relevant check to fail, then remove those probes and rerun the checks.
- Confirm a temporary file under `design/magicpath/` is omitted by lint, format, typecheck, and test; remove it afterward.
- Inspect the diff for overwritten documents, generated Git metadata, environment values, and unrelated files.

## Builder decisions

- Use the generator's versions at implementation time rather than pinning versions in this design. Review the resulting `package.json` and lockfile before installing.
- Keep the starter page minimal; product layout belongs to the later UI specs and MagicPath prototype.
- Sonar project identity stays as TODOs as required by the spec. The quality gate can be checked once those identifiers and a PR integration exist.
