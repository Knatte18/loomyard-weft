```yaml
lens: board-fit
outcome: reported
```

Board-fit review of the validator-checks decision record against the board, its peer runs and parent commits.

**Findings**

1. MEDIUM, consistency, `## Open risks` bullet 2. The risk that `review-setup-tuning`'s repository-invariant lens may still claim seam, tier and `doc.go` catches as its own is already settled by that peer's record.
   - Evidence: its record says "`validator-checks` (running) adds machine checks for seam tests, tier tags and `doc.go` lines; `repo-invariants` names no check of its, so the lens stays right whether or not those checks land first".
   - Its Scope Out also says new checks in `validate-discussion` and `validate-plan` "belong to `validator-checks`".
   - Fix: drop the bullet, or turn it into a citation of that line.
2. LOW, design, D5 (the marker census and scan). `landing-settles-in-place` is in Webster and changes `internal/loomcli/landingdeps_test.go`.
   - That file holds the reflective `landingDeps` drift-guard test, which goes into D5's census of tests to mark.
   - Its record's plan notes say the test "changes with D2 and with D3's `EscalateJudgedFirst`".
   - Whichever run lands second meets the other's version of that test: either it is marked here against a changed shape, or a newly written reflective test arrives without `//lym:seam` and fails the scan at the merge.
   - Fix: name in Open risks that D5's census runs against the parent at Plan time and that a peer adding a reflective test before landing needs the marker. This is an ordinary misalignment the writer can settle, not a parent question.

**What I checked**

- `review-setup-tuning` (T3, running): overlap on the seam, tier and `doc.go` theme, settled by its own record (finding 1). PATTERN entries don't conflict: it changes the `PATTERN-config-layers` background and this record changes `PATTERN-test-speed`.
- `landing-settles-in-place` (T2, Webster): text overlap on `internal/loomcli/wiring.go` and `landingdeps_test.go` (finding 2). Its PATTERN change is `PATTERN-shed-stop-chain`, so none there.
- `creel-hook-post` (T1, Plan review): no conflict. It cites `PATTERN-test-speed` only as a constraint (an LLM-tier placement) and doesn't change it.
- `turn-batching` (T5, depends on this task): it later adds sentences to the Discussion and Plan writer stencils. That is sequential, and D10's pointer sentences fit beside them, so no conflict.
- Parent commits: `lym loom peers` lists none.
- Decision notes: the board list has no `Decision for validator-checks:` note, and no `Added after Discussion` entry exists, so there was none to read.

**Tool calls**

- Grepped the `landing-settles-in-place` and `creel-hook-post` decision records for `PATTERN-test-speed`, the validators, the specs and `PATTERN changes`. Context held neither record's body.
- Read `landing-settles-in-place`'s `## PATTERN changes` and plan notes, and the head of `review-setup-tuning`'s record, which had not existed at exploration time.
- Grepped `review-setup-tuning`'s record for seam, tier, `doc.go`, validator and PATTERN lines to test for overlap. Context lacked its body.
