```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code paths that already touch the state each decision changes. I found 2 MEDIUM findings and 3 LOW findings.

**MEDIUM — D11 (rename) vs reports over archived runs.**
- Summary: renaming `loomshed.NameMergeIn` makes the reports misfile archived `Merge-In` rows. This undercuts D4's own aim of keeping reports over archived runs correct.
- Evidence: `internal/reportengine/wakes.go:106` has `VerifyRows: []string{loomshed.NameMergeIn, retiredPublishRow}`. `internal/reportengine/timingspans.go:74,104` map `NameMergeIn` to `PhaseMerge`.
- Effect: archived statuses and traces name `Merge-In`. Once the constant reads `Parent-Align`, those rows drop out of the merge phase and the verify-row counts, so timing and wake reports change for every window before the landing.
- Precedent: `retiredPublishRow` shows how to keep a retired row name for archived runs. D11 says nothing about it.

**MEDIUM — D5 (perch fan) vs the Burler cluster-fan contract and its reading manifest.**
- Summary: `goal-scope` and `board-fit` are emphasis lenses. The record drops them into `review-discussion`, a production fan, without saying how they fit its procedure-lens contract.
- Convention: the header of `internal/lenses/template.yaml` says each review-* and fix-* production fan's lenses are procedures that end in their own list.
- Contradiction: `contracts/stencils/burler/burler-cluster-fan.md` tells each fork "Report every finding your lens's procedure produces and nothing outside it". Both lenses end "Report anything else you notice too — emphasis, never exclusion."
- Context gap: the same stencil says the fork's inherited context already holds the target, the rules and the code from the reading manifest. That manifest (`burler-fork-reading.md`) names no board or `lyx loom peers` output.
- Effect: D6's reads (`lyx board list`, `lyx board get` per touching entry, `lyx loom peers`, `lyx loom peers show`) must all fit in the fork's cap of 6 tool calls. Otherwise the review lead must read them before it forks, which the record does not say.
- What the record needs: either rewrite both lenses as procedures ending in a list, or add the board and peers reads to the Discussion segment's reading manifest.

**LOW — D8 (decision escalation) vs the Bouncer's judgment refusal.**
- Summary: a BLOCKING `decision` finding, such as a plainly outdated entry, makes the Bouncer refuse any CONVERGED verdict before D8's rule runs. The refused round is re-judged at one bounce unit per row.
- Evidence: `internal/shedadapters/bouncer.go` `judgmentRefusal` refuses CONVERGED when a review finding is BLOCKING. Per `shedadapters/doc.go`, a refused judgment is archived and re-judged through the unjudged-round hand-back "at one bounce unit on each row".
- Effect: "whatever the verdict" in D8 does not cover the refused-CONVERGED path, so the escalation can come late and spend budget.
- Mitigation: round 1's rule already makes the judge rule CONTINUE over any BLOCKING finding, so this path is rare. The record should still say D8 runs on the refused path too, or that it does not.

**LOW — D11 vs runs already past Merge-In.**
- Summary: the Code-Perch's review input is a scratch file named after the row. A run at Code-Review or later when the binary changes loses it.
- Evidence: `internal/loomcli/subjects.go:235` reads `loomshed.NameMergeIn+"-fixer-report.md"`.
- Effect: D11 restarts only runs parked on `Merge-In`. A run past that row is not restarted, and after the rename its fixer report is read under `Parent-Align-fixer-report.md`. The Code-Perch would miss the latest merge fixer report and its review-input timestamp.
- The record should state that this is accepted, or keep reading the old name.

**LOW — D8 vs every reader of the escalation cause.**
- Summary: the cause set is hard-coded and checked in more places than the notes name. The new `decision` cause would be refused on write and on read.
- Evidence: in `internal/shedadapters/escalation.go`, `EscalationCause.valid` accepts three values. `writeEscalation` refuses anything else, and `ErrEscalationMalformed` covers a record with another cause. `internal/shedadapters/circling.go:222` refuses a decision file whose cause is not one of the three. `internal/arbiterstop/arbiterstop.go:77` switches on cause.
- Effect: the record's notes point at the docs and the `lyx loom circling` help, not these validators. Plan will meet them anyway, so this is a pointer gap only.

**Paths traced, by decision:**
- D1: alignment importers in `loomcli`, `orchengine` and `reportengine`; `DecisionNoteTitle` callers — survives.
- D2: batten's `ReviewWait` seam (named in the record); the gate's expiry probe in `entries_gate.go`, which goes with the gate; an in-flight parent-review wait at upgrade — survives as the record states.
- D3: reconcile, and read-time migration of profiles and `run_settings` (fc75aef79 precedent) — survives.
- D4: `reportengine` parse of `wakes`, `runs` and `timingstops` — survives. Row-name classification breaks under D11 (first finding).
- D5: `MaxFanLenses` (16), the `review-discussion` fan length, and this hub's override of `discussion_reviewer_fan` — survives. The cluster-fan contract and fork context break (second finding).
- D6: `lyx loom peers` access from a fork (no shell-guard row blocks it) — survives, subject to the D5 finding.
- D8: the Call-entry clear, which skips an escalated or decided round (`bouncer.go:347`) — survives. Recorded-decision handling (`bouncer.go:1194–1197`) and the arbiter switch — survive. `judgmentRefusal` — partial (LOW). Cause validators — need extending (LOW).
- D10: shuttle's hold on a turn end without every output file; `decision add` refusing a darn run before its record — survives.
- D11: `start.go:165`, the interrupt policy table, the gateslot `landing` keys and `goReadGates`, and the envkit test fixtures — rename-consistent. `reportengine` breaks (MEDIUM). The scratch fixer-report name breaks for runs past the row (LOW).

**Tool calls:**
1. Grepped escalation cause constants and their readers, to find where the cause set is checked; the context lacked the cause readers.
2. Read `shedadapters/escalation.go` and the cause readers, for `valid()` and its callers; the context lacked the validation code.
3. Grepped the Bouncer `Call` entry and clear ordering, to see whether an escalation over a CONVERGED verdict survives the clear; the context lacked the code.
4. Read the clear-skip rule, `judgmentRefusal` and the settle switch, to confirm the clear skips escalated rounds and to find the BLOCKING refusal.
5. Grepped `Merge-In` row and gate-key readers outside fabric, to find report and scratch-file readers of the row name.
6. Read `MaxFanLenses` and the Burler fork tool-call cap, to check the fan length limit and what fork contract D5's lenses enter.
