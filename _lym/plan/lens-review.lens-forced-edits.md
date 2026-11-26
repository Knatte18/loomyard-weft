```yaml
lens: forced-edits
outcome: reported
```

Lens forced-edits: find what each card's changes force outside the members it names.

- **LOW, Card 4 / D7: the orch-role prefix rule is not listed as a change.** `contracts/orchrole/orch-role.md:81` says "Each mail body or line whose text starts with `[batten notice]` or `[dobby notice]` is a notice." Card 4's Intent adds `[run watch]` only to the "Only these reach you" list. If line 81 is left as it is, the orch still reads a `[run watch]` line as no notice, which is the gap D7's rationale names. Fix: Card 4's Intent says line 81 names `[run watch]` beside the two prefixes.
- **LOW, Card 4 / D7: a stale copy of the orch role outside the repo.** The hub stencil `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/orch/orch-template-role.md:44,52,91` lists the notice prefixes and conditions too. No file in the repo has that name; `contracts/orchrole/orchrole.go:12` embeds only `orch-role.md`. No card can reach that copy under `card-fabric-reference`. Fix: record in `## Shared Decisions` that the hub copy is out of the plan's reach.
- **LOW, Cards 4 and 7: the widened test's comments go stale.** The file comment of `internal/beamcli/dobbynoticestamp_integration_test.go:1` and the doc of `internal/beamcli#TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy` both speak of run notices only. After Cards 4 and 7 the test also posts the watch line and the gate-hold notice, and asserts that nothing is superseded. Neither card says the two comments get rewritten. Fix: Cards 4 and 7 state the new scope in that test's doc comment; the file comment changes with it.
- **LOW, Card 1: a test file comment goes stale.** `internal/dobbyengine/tasksetting_test.go:1` says it "pins the per-task lookup of beam.yaml's pull_request, land_notice and autonomy". Card 1 adds a `run_watch` subtest to `internal/dobbyengine#TestLookupTaskSettings` but does not name this comment. Fix: Card 1's Intent names the file comment.
- **LOW, Card 5: the gateslot package doc does not name the marker.** `internal/gateslot/doc.go:11,31` describes the Holder record and a tier run's environment. Card 5 adds `Marker` to `internal/gateslot#Holder` and lists neither the doc nor any line naming the marker. Fix: Card 5 adds a doc line saying a tier run's record carries its environment marker.
- **LOW, Card 1: an unverified gap in the board's run_settings reader.** `internal/boardengine/store.go:404` reads only `pull_request` and `land_notice` from a board entry's `run_settings` into `EntryBeamSettings`. A task setting `run_watch` there is not carried by that reader. I did not trace what consumes `EntryBeamSettings`, so it may not matter for D2's chain, which reads the worktree's local layer. No card says either way.

Tool calls beyond context:
1. `lym quarry resolve` on the glyphs fork0 named that the exploration never resolved, to check them. All were found.
2. Grep for `adopt_fix_tries` across the repo, to look for a beam template golden or doc that Card 1's template change would break. None found.
3. Read `internal/boardengine/store.go:625-655`, the reinterpret test line, and the per-task key mentions in md files, to look for other per-task key lists.
4. Grep `role-beams.md` and the board skill for per-task key lists, and the beamcli tests for reflection over Deps fields or an import allowlist. Neither is forced.
5. Grep for readers of the notice prefixes and for gateslot's Holder docs, to find prefix readers and record docs the exploration lacked.
6. Searched for the source of `orch-template-role.md`, to learn whether a repo card could reach it.
