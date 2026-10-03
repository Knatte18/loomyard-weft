# Support log — test-suite-change-amplification

## Interview

No operator answered questions in this session; every design point was settled by auto-pick from the board entry, GitHub issues #323–#325 and a coarse read of the code.

1. Board read.
   The task carries #324 and #325 in full; #326, #327 and the tmux socket leak go to `test-suite-derived-guards`, which builds on this task's `internal/testkit`.
   `pattern-init` later moves `CONSTRAINTS.md`, so the Testkit Invariant lands there first.
2. Drift check against HEAD (31 commits past the survey's `fafb660c3`).
   The old-policy comments, the `shuttleengine.Engine` fake copies, the `newFabricFixture` users, the reedengine `execHook` installations and the `buildLyxBinary`/`containsPath`/`equalStringSlices` copies all still exist.
   `internal/loomshed` is new since the survey and holds some of the loom fixtures; it carries one old-policy citation the issues did not list.
3. Kit placement (D1, D2).
   Took #324's top-level `internal/testkit/<kit>/` rule; moved #324's `internal/output/outputtest` under it as `internal/testkit/envelope` so one invariant covers every shared kit.
4. `buildLyxBinary` (D3).
   It spawns `go build`, which a spawn-nothing invariant forbids.
   Kept the item in scope with one named, bounded exemption (`lyxbin`), its token banned from untagged files.
5. Tier Purity for new gitkit helpers (D4, first pass).
   First draft followed #325: one banned token per new spawning export.
6. Operator directive arrives mid-session.
   The orchestrator session relayed that the operator authorizes changing `CONSTRAINTS.md` invariants that grew the suite; confirmed by re-reading the board body, which now carries the directive.
7. Invariant audit (D13).
   Read the invariants bearing on #324/#325's tests.
   Found `cmd/lyx/tierpurity_test.go`'s `bannedTokens` and `cmd/lyx/hermeticenv_test.go`'s `gitSpawnTokens` each hand-keep a list of spawning gitkit helpers, already disagreeing (`MustRun`/`SeedConfig` in one only, `GitStatusPorcelain` in neither).
   D4 rewritten: one shared rule, "every gitkit export but `HermeticGitEnv` spawns git", used by both scans.
   This tightens Tier Purity; untagged gitkit callers in `fabricengine` and `websterengine` move to tagged files.
8. Old-policy citations beyond #324's list.
   `loomcli`'s validate and parity tests and `loomshed`'s fixture copy glyph-plan and file-tree writers from `planglyph` and `loomshed`.
   First draft added `internal/testkit/plankit` to share them; parent review round 1 (F1) showed #327 section 2 already owns that writer in `test-suite-derived-guards`.
   Final: the fixtures stay put, their citations are reworded without converting the code, and the sites are noted for part 2.
9. Reflective wiring test (D8): `intentionallyNil` bounded to one field path and reason per entry.
10. Docs (D11): `testkit` is not a CLI module, so no `manifest/designs/` doc; `docs/overview.md` already describes `gitkit`/`hubforge` and gains `testkit`.

## Rejected alternatives

- Kits nested under the package they fake (e.g. `internal/shuttleengine/shuttlefake`): the per-package seam scans would scan them as production code of a bound package.
- `internal/output/outputtest` for the envelope decoder: outside `internal/testkit`, so the Testkit Invariant would not cover it.
- Dropping #324 §6's `buildLyxBinary` item to keep the spawn-nothing rule exceptionless: leaves four copies; the bounded `lyxbin` exemption costs one named line.
- Putting `BuildLyx` in `gitkit`: `gitkit` is git primitives only.
- One Tier Purity token per new gitkit export (#325's own proposal): two hand-kept lists across two scans, the exact mirror the operator directive targets.
- A gitkit allowlist of spawning exports shared by both scans: still a list to edit per helper; the inverse rule (all but `HermeticGitEnv`) needs no edit when a helper is added.
- Banning `MustRun` and friends but allowlisting today's untagged callers: keeps Tier 1 spawning git under an exemption instead of tagging the files.
- A new scan forbidding raw git spawns in `_test.go` files outside `gitkit`: belongs with part 2's scan harness; recorded as review discipline here (D6).
- Moving the Hermetic Env per-package `TestMain` into some global hook: `go test` has none, and the five-line file never churns.
- Folding the per-package seam/leaf scan scaffolding into one harness now: that is #326, part 2's scope.
- Placing the Testkit scan in `cmd/lyx` beside the repo-wide guards: `cmd/lyx` is the worst churn package, and part 2 will host scans under `internal/testkit`.
- An `internal/testkit/plankit` sharing the glyph-plan and file-tree fixtures: duplicates #327 section 2's plan-fixture writer, which the board assigns to `test-suite-derived-guards` (parent review F1).
- A `manifest/designs/testkit.md`: `testkit` is not a lyx CLI module; package docs and `docs/overview.md` cover it.

## Review rounds

### Parent review, round 1

Verdict BLOCKING (`_lyx/reviews/parent-review/round-1/review.md`).

- F1 [BLOCKING] `plankit` duplicates #327 section 2's plan-fixture writer — fixed.
  `plankit` is dropped from D2, Scope and the acceptance criteria; the glyph-plan and file-tree fixtures are listed out of scope for #327; D1 rewords their old-policy citations without converting the code; the acceptance criterion on citations says so; D13 notes the sites for `test-suite-derived-guards`; Q7 and interview item 8 are updated.

## Question ledger

- Q1 Where do shared kits live? Resolved, auto-pick: `internal/testkit/<kit>/` (D1).
- Q2 Where does the JSON envelope decoder go? Resolved, auto-pick: `internal/testkit/envelope` (D2).
- Q3 Is `buildLyxBinary` shared despite spawning? Resolved, auto-pick: yes, as `lyxbin`, the one bounded spawn exemption (D3).
- Q4 Where does the Testkit scan live? Resolved, auto-pick: `internal/testkit` (D3).
- Q5 How does Tier Purity learn about new gitkit helpers? Resolved, auto-pick after the operator directive: one shared "all of gitkit but `HermeticGitEnv`" rule for Tier Purity and Hermetic Env (D4).
- Q6 Do the #325 "rules going forward" become scans? Resolved, auto-pick: review-discipline clauses in `CONSTRAINTS.md`; a scan is part 2's (D6).
- Q7 Are the loomcli/loomshed glyph-plan fixture copies in scope? Resolved after parent review round 1: no; #327 section 2 owns them in `test-suite-derived-guards`, and this task only rewords their old-policy citations (Scope, D1).
- Q8 How is the test count compared? Resolved, auto-pick: leaf tests from `go test -json`, every drop itemised (Acceptance criteria).
- Q9 Which invariants change under the operator directive? Resolved, auto-pick: D13's audit; none weakened, Tier Purity tightened.
- Q10 Does `testkit` need a `manifest/designs/` doc? Resolved, auto-pick: no (D11).
- Environment note: no tool failure occurred; `lyx board get` and `gh issue view` worked.
