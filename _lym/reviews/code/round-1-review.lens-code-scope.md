```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the board entry's goal points.

**Decisions** (a decision record alone, so these are the goal points under `## Goal`; the record has no `Added after Discussion` entries):
- G1 **Takes**: (a) the ledger write in `landLocally` / `recordHeldLanding` takes the slug, tip and records commit as arguments, with the records commit picked by `oldestHeldRecords`; (b) `Deps.LandLocally` takes the run's own landing commit, which a lone run's Finalize passes; (c) a standing landing is rewritten with the same tip and keeps its records commit under the stated condition, and the call still commits the status records and marks the pair's push; (d) no queue, no batch, no leader.
- G2 **Delivers**: one writer that takes another run's values, with single-run landing unchanged; a parent ahead by a chain of runs' own tips reads as ahead by owed tips alone.
- G3 **Exports**: the loomcli writer taking the parent branch, slug, landing commit and records commit, plus the landingshed Deps seam; the change description lists both signatures.

**Changes → decision**
- C1 `internal/landingshed/finalize.go:566-573`, `:958-975`, `:1026-1058`: one head read after the merge, handed to `recordLanding` and `pushParent`/`LandLocally` → G1(b).
- C2 `internal/landingshed/deps.go:173-175`: `LandLocally` doc says tip is the run's own landing commit → forced by C1.
- C3 `internal/loomcli/landingdeps.go:949-985`: `writeLedgerLanding(parent, slug, tip, records)` extracted, and `recordHeldLanding` keeps `oldestHeldRecords` and calls it → G1(a), G2 (writer), G3 (writer).
- C4 `internal/loomcli/landingdeps.go:922-923`: `landLocally` doc update → forced by C3.
- C5 `internal/loomcli/landlocally_integration_test.go:467-519`: the chain-of-three test → G2 (owed-tips read) and G1(c), the re-record keeping its tip and records commit; this is the record's acceptance criterion.
- Commits be85ce0a7 and e38f42903: comment rewraps of C3 and C4 → forced.

**Decision → changes**
- G1(a): C3. G1(b): C1, C2. G1(c): no new code; the existing `oldestHeldRecords` path is reached through C3 and exercised by C5. G1(d): no change adds a caller that records another run, which holds.
- G2: C3, C5.
- G3: C3 for the writer and C1/C2 for the seam; `_lym/landing/summary.md` lists both signatures. There is no plan, so "the plan names both" has nothing to check. Partly built, see F1.

No change traces to nothing: no unrequested scope. No goal point is left unbuilt.

**Findings**

F1. Severity: LOW. Class: scope. Location: `internal/loomcli/landingdeps.go:979` (`writeLedgerLanding`).
- Summary: G3 fixes the exported writer as taking the parent branch, but `writeLedgerLanding` takes a `runpusher.Target` and reads the branch itself with `CurrentBranch()`. The signature fixed for landing-queue-batch therefore differs from the one the entry names.
- Evidence: `decision-record.md:7` says the writer takes "the parent branch, the run's slug, its landing commit and its records commit". The diff `002-internal-loomcli.diff:38-44` shows `writeLedgerLanding(parent runpusher.Target, slug, tip, records string)` reading `gitrepo.New(parent.Location.WorktreePath()).CurrentBranch()`. `summary.md:35-36` gives the writer's own reason, which is a claim and not the measuring stick.
- Fix: take the branch as an argument next to the target, which `runpusher.RecordLanding` still needs. Or, if the target is judged to be "the parent branch", the parent rules on it.

Tool calls: none; the inherited context held everything this lens needed.
