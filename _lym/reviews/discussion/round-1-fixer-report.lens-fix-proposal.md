```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F6) and look for cases where the proposed change is wrong.

**F1, MEDIUM: the emphasis line contradicts `board-scope`'s "reports only" contract.**
- Changed text: D1's new bullet "Ends with the line 'Report anything else you notice too — emphasis, never exclusion.'". The D1 acceptance line now says the prose "reports only contradictions with the board, … and ends with the emphasis line".
- Evidence:
  - D1 still says "Reports only a contradiction with the board" and "Excludes text-only overlap, shared bookkeeping lines and any question a decision note … settles".
  - The proposal makes the same prose both restrict its reports and invite anything else. The plan writer gets two opposite instructions, and the acceptance line asserts both.
  - Rationale D1 says the lens exists because the pair was noisy and "reports contradictions only". The emphasis line reopens exactly that noise.
- The review offered a second route that avoids this: widen the template header and the `config_test.go` procedure-lens exemption to name `board-scope`. The fixer took the first route. Either the record reconciles "reports only" with the emphasis line, or it switches to the exemption route.

**F2, LOW: Goal and Scope still name only `light` and `docs` after `speed`'s loom block joined D6.**
- Changed text: the D6 heading, D6's first sentence and D7's first bullet now include `speed`'s loom block.
- Evidence:
  - Goal: "a solo code review in the `light` and `docs` profiles".
  - Scope In: "The `light` and `docs` profiles in `internal/runprofile/template.yaml`".
  - Neither names `speed`, so the record's scope no longer covers a change D6 makes.

**F3–F6: no case found where the proposal is wrong.**
- F3: the two-hop rename `webster_fan` → `webster_reviewer_fan` → `code_reviewer_fan` (config.go:1022 and :1035) matches the "same bound" claim.
- F4: the dropped sites hold no closed-to-local text (grep shows none).
- F5: `fanKeys` covers only the six reviewer and fixer keys.
- F6: wording only.

No further tool calls made.
