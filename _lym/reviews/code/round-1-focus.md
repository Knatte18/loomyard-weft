---
round: 1
focus:
  - "internal/hubforge/template.go, per-key lock and shared map lock: do two keys build concurrently, does a key build exactly once under parallel first callers, and can the map lock be held across a build or taken in an inverted order?"
  - "internal/hubforge/template.go, failure marking: does a failed build return its own error to the first caller and a key-naming error to every later caller without rebuilding, including when the build panics, calls FailNow or skips?"
  - "internal/hubforge/template.go, registration identity: are two different registrations with one key refused naming the key, while the same registration used twice is accepted?"
  - "internal/hubforge/template.go, refusing testing.TB: do Cleanup, Setenv, TempDir, Chdir and ArtifactDir each fail naming the key, and does any other TB method let template state escape the template root or outlive the build?"
  - "internal/hubforge/template.go, upstream-tip check: is the tip read from the records bare for the board's current branch, never from the board's tracking ref, and does a commit pushed into the bare from a clone inside the template root fail the build naming the key?"
  - "internal/hubforge/template.go, build order: is the template built in its own lymtemp root from the bares, clone and base shape pairs, then the build function, then D3's pack and the fingerprint, and never from a copy?"
  - "internal/hubforge/template.go, shapes now built through registrations: does the shape copy path keep its earlier relocation, rebase and fingerprint behaviour, and does the closed Shapes set still resolve alongside the lookup by key?"
  - "internal/hubforge/template_test.go, test economy: do the new tests cover each acceptance criterion without duplicating hub_test.go or another test, and does any new behaviour in template.go lack a test?"
  - "pattern/PATTERN-test-isolation.md and internal/hubforge/doc.go: does the pattern diff carry the declared change (hub fixtures via hubforge, the background's hub-fixture paragraph gaining the registered template), and do the doc.go claims match the code?"
  - "Commit messages and the change description: does any claim in them, such as the build's refused TB methods or skips, go beyond what the diff does?"
---

Seed focus for round 1 over `git diff 7ebd7fc54648b0b2428780f5723465cdf636483c..HEAD`.
The answer key is the decision record alone, D2 and its acceptance criteria.
The diff touches `internal/hubforge`, `pattern/PATTERN-test-isolation.md` and one line of `hub_test.go`.
