# Specs index

`specs/NNN-*.md` define WHAT (requirements and acceptance criteria) and are owned by
the human. Superpowers writes the HOW: design docs in `docs/superpowers/specs/` and
plans in `docs/superpowers/plans/`. One thread and one branch per spec.

Execution order (004 runs second because the UI specs depend on the prototype):

| Order | Spec                                     | Status |
| ----- | ---------------------------------------- | ------ |
| 1     | 000-tooling                              | done   |
| 2     | 004-design (MagicPath prototype + theme) | todo   |
| 3     | 001-feed                                 | todo   |
| 4     | 002-post-message                         | todo   |
| 5     | 003-health                               | todo   |

Update the Status column (todo / in progress / done) in the PR that changes it.
