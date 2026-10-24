---
round: 1
focus:
  - "internal/cliwire/paths.go, repositoryRootBelowCeilings: does the ceiling comparison match git's semantics (a ceiling is never climbed into, the start directory is checked even when listed), and does it compare like with like when `dir` and the ceiling spell the same directory differently, such as a symlink, a trailing separator or Windows case and volume forms?"
  - "internal/cliwire/paths.go, gitCeilingDirectories: do the split, the absolute-only filter and standalonestate.Normalize drop or keep entries as the decision record says, and can Normalize fail or change an entry so a legitimate ceiling no longer matches the walk's candidates?"
  - "internal/testkit/tmuxkit/tmuxkit.go, setTempEnv: does the ceiling equal the directory t.TempDir builds under on every platform and for a re-executed helper binary, and can setting it break a test that needs git discovery above the run directory?"
  - "Tests: does the new cliwire_test.go row cover both acceptance cases (a `.git` above the ceiling is missed, a `.git` below it is found) through the unexported walk without t.Setenv, and does the TestSetEnv change duplicate or weaken existing coverage?"
  - "Docs and conventions: do the RepositoryRootOf doc and the tmuxkit package doc carry the ceiling rule as the record's plan note asks, and do the touched Go comments follow the repository's one-sentence-per-line rule and PATTERN-cwd-resolution (no cwd query added)?"
---

Diff is small (four files); the correctness risk sits in path comparison between the walk's candidates and the normalized ceilings.
