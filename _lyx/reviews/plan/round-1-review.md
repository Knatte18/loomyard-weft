---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/02-orch-rules-verb.md
    summary: card 2 adds the operator verb lyx orch rules but no card updates the orch module line of docs/overview.md, which lists the orch verbs as start|status|refresh|distill|stop|cleanup
    fix: "In card 2, add the bullet `//docs/overview.md` under **Edit:** after `//internal/orchengine/doc.go`, and add one Intent sentence saying the orch module line of `docs/overview.md` (the `lyx orch start|status|refresh|distill|stop|cleanup` list) gains `rules`, giving `lyx orch start|status|refresh|distill|stop|cleanup|rules`. Checked with validate-plan --overlay; it adds no blocking finding."
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:way-forward
    location: _lyx/plan/02-orch-rules-verb.md
    summary: the push refusals of writeHubRules steps 2 and 4 name a re-run as the way forward, which fails every time once Push's recovery rebase conflicts with an upstream change to the rules file
    fix: "In card 2, `writeHubRules` steps 2 and 4: when the `Push` error wraps `gitrepo.ErrPullRebaseFailed`, the refusal names the board's diverged recovery in place of a bare re-run: in the board dir, run `git pull --rebase` and resolve the conflict, then `lyx board sync`, then re-run `lyx orch rules --from <file>`. Every other push error keeps today's re-run wording. Add one `TestWriteHubRules` row whose fake first `Push` returns an error wrapping `gitrepo.ErrPullRebaseFailed`, and assert the refusal names `git pull --rebase` and `lyx board sync`, with no write call and the file unchanged."
  - id: F3
    severity: LOW
    class: scope
    origin: lens:affected-set
    location: _lyx/plan/01-rule-files-render.md
    summary: card 1's new row set for TestRenderFiles_WriteWithoutLeadingComment drops the role row, which today is the only assertion of the decision-note title fill (Decision for <slug>:)
    fix: "In card 1's Tests bullet for `TestRenderFiles_WriteWithoutLeadingComment`, say that the both-present row keeps `want: []string{\"## Commands\\n\", testIndex, \"Decision for <slug>:\"}` beside its `wantOrder`, so the index and decision-note title fills stay asserted."
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/02-orch-rules-verb.md
    summary: fakeRulesBolt never skips on its unpushed count, so no row reaches the state the first push exists for, and the claim that dropping either push fails the retry row holds only through the call-log assertion
    fix: "In card 2's `fakeRulesBolt` spec, add a seedable `upstreamMoved` flag: `PullThenCommitWritten` returns a `diverged` skip without running `write` when `upstreamMoved` is set and `unpushed > 0`. The re-run-after-failed-push row seeds `upstreamMoved`, so a dropped first push fails that row on its exit code and fields as well as on the call log. Reword the row's closing clause to say which assertion catches each dropped push."
---

### [MEDIUM] docs/overview.md keeps the old orch verb list

**Location:** `_lyx/plan/02-orch-rules-verb.md`; evidence at `docs/overview.md:392`.

**Issue:** The orch module line names `lyx orch start|status|refresh|distill|stop|cleanup`, prime only.
Card 2 adds `rules`, an operator verb, but edits only the two packages' `doc.go`.
`CLAUDE.md`'s "Docs land in the same commit" rule requires `docs/overview.md` to change in the same commit when the module table changes, and the orch line's verb list is part of that table.
Every other member of this change's affected set is covered: `Command`, `TestOrchIntegration_Refusals` and its file header, both `doc.go`s, and the generated index.

### [MEDIUM] Push refusals send the caller into a loop when the rebase conflicts

**Location:** `_lyx/plan/02-orch-rules-verb.md`, `writeHubRules` steps 2 and 4; evidence at `internal/fabricengine/bolt.go:82-91` (`PushRecorded`) and `bolt.go:314-316` (`divergedDetail`).

**Issue:** Take the case where Bolt holds the verb's own rules commit unpushed and the upstream has changed `_lyx/orch/rules.md`, for example after a run from another machine.
`Push`'s recovery rebase then conflicts, and `PushRecorded` drops only seed commits, so `Push` returns the rebase failure.
Step 4 names "the next run of the verb" as the way forward, and step 2 names "a re-run once Bolt pushes".
Every re-run fails again at step 2, before the pull-first write runs, so neither step can succeed from that state.
D3 settles only the non-conflicting rebase ("whose recovery rebase carries them over a non-conflicting upstream change") and leaves this case to the plan.
The board already words the recovery for this state, in `divergedDetail`.

### [LOW] Only assertion of the decision-note title fill is dropped

**Location:** `_lyx/plan/01-rule-files-render.md`, the Tests bullet for `TestRenderFiles_WriteWithoutLeadingComment`; evidence at `internal/orchengine/prompt_test.go:37`.

**Issue:** Today the `role` row asserts `## Commands\n`, `testIndex`, `## This repository\n` and `Decision for <slug>:`.
Card 1's listed rows leave out the both-absent row, which is this one, and no listed row keeps its `want` strings.
A grep of the orch test files finds no other assertion of the decision-note title in the rendered role file.
`testIndex` is still asserted elsewhere (`resumecontext_test.go:75`, `watcher_compact_test.go:252`), so the decision-note title fill is the only coverage lost.

### [LOW] The fake cannot reach the state the first push guards

**Location:** `_lyx/plan/02-orch-rules-verb.md`, Tests (`fakeRulesBolt` and the re-run-after-failed-push row); evidence at `internal/fabricengine/bolt.go:284-298`.

**Issue:** D3 gives the first push one reason: without it, `pullFirst` would skip a Bolt that holds an unpushed rules commit behind a moved upstream as `diverged`.
The fake's `PullThenCommitWritten` returns only a configured skip, so that state never arises in a row.
Its `Push` zeroes `unpushed` whichever push succeeds, so the retry row still ends with `unpushed` 0 when the first push is dropped.
The row's "dropping either push fails it" holds for the first push only through the call-log assertion.

## Rejected

- way-forward, LOW: the `--from` read, pre-write and write-failure refusals name no next step. `PATTERN-refusal-way-forward` binds webster, shed and loom refusals, not orch ones. D3 fixes these refusals' content (naming the flag and the file, and "unchanged, nothing written"), and the card follows it.
- way-forward, LOW: `readRules`' "fix the file or remove it" sends the operator to hand-edit Bolt. D2 keeps today's read-error behaviour ("any other read error fails the render naming the file, as today"). The state that triggers it is a directory or a permission fault at the path, which comes from local filesystem damage rather than a content edit, so it is not the hand edit D3's rationale guards against.
- guard-reach, LOW: under `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` the envelope reports `unpushed: false` for a commit that was never pushed. Those variables are the test and CI bypass gate. `PullThenCommitWritten` ignoring `SyncOptions` is existing `fabricengine` behaviour that every pull-first board writer shares, and this plan neither introduces nor licenses changing it.

## Focus verdicts

- Card 2, attack surface: the Intent states what the verb lets through (any `--from` content published to Bolt and its remote) and what bounds it (one Bolt path, one read file, prime-only cwd, the role preamble). Every refusal branch has a named input and a `TestWriteHubRules` row or the integration refusal step that reaches it. Not a defect, apart from F2's way forward and F4's fake.
- Card 2, CLI shape: a leaf with the operator audience, `cobra.NoArgs` and a `--from` flag that cobra does not mark required, so `Use` is the bare name. The audience bullet admits it. Not a defect.
- Cards 1 and 2, ordering: card 2's `rulesRel` and `orchPaths` fields come from card 1, and card 3's Verify needs cards 1 and 2 done. No card depends on a later one.
- Cards 3 and 4, board entry: neither edits this task's own entry. Card 4 writes the new entry `orch-rules-move`, a note with one type label.
- Fidelity: D1–D5 and every Constraint are carried. The extra `## Rules files` heading for the preamble is a faithful reading of D2's "moves above the two sections".
- Verify coverage: cards target only `internal/orchengine`, `internal/orchcli`, `contracts/stencils` and non-package files, and `## verify:` runs all three packages in both tiers. F1's fix adds a `docs/` file, whose link check runs in `internal/lyxcwd`, outside this rule's package scope.
- Test economy: no planned test duplicates an existing one. The heading strings in `wantOrder` anchor Go-filled markers' positions, which `PATTERN-test-economy` allows.
- `Edit` member glyphs and arrows: `RenderRoleFile` and `renderRole` carry the arrow. `orchengine.Paths` is a struct, whose field changes take no arrow. Not a defect.
