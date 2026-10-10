---
round: 3
focus:
  - "D12: the `.git` entry reader that `lyxcwd` is to import. Check the import graph of every package that compiles it, in production and in tests, including `gitrepo`'s in-package `integration` test files that import `gitkit`, and settle whether any path leads back to `lyxcwd` so that Go rejects an import cycle."
  - "D12: the walk-up's definition of a usable `.git` entry. Settle whether it matches `git rev-parse --show-toplevel` for a gitfile whose `gitdir:` target is missing and for an empty `.git` directory inside an outer repository, and whether the acceptance criterion pins both cases."
  - "Constraints, `PATTERN-cwd-resolution`: settle whether the constraint states every pinned import cap and background line that D12's new `lyxcwd` import changes, and any new standing rule on the reader package's imports."
  - "D8 and Scope: settle whether the `/proc` probes shared by the watchdog recognizer and the leftover scan end up in one copy in `tmuxkit`, as `PATTERN-testkit` requires."
  - "D12 and D3 acceptance criteria: settle which census column the `rev-parse` drop is read from, given that the code-side split is approximate wherever hub builds overlap other tests."
  - "Constraints, test budget: settle whether the rule for rising `test-budget.yaml` rows names a rule or an enumerated package list, and whether any package whose tests D6 or D8 split is missing."
---

The entries restate the open design questions without naming prior findings.
