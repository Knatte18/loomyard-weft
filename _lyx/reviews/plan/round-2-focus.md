---
round: 2
focus:
  - "Card 9 adds a fifth recipe row `Land` to `internal/battenrecipe` and a `Land` key to the `internal/shedrecipe` registry. Settle whether every existing test that asserts the recipe's row count or names, the registry's `Names()` list, or a `minimalEntries` row per registered name, or that steps the embedded recipe through to Worktree-Teardown in `internal/battencli`, is a target of some card."
  - "Shared Decision `land-row-pause` and card 8's step 1 say a pause during Land's lock or dirty-prime wait ends in a Stuck naming `lyx batten run <slug>`, which re-runs the row from the top. Settle whether `internal/shedengine` clears `PauseRequested` on a blocked Stuck, and whether a test in the plan drives a pause during Land and then the named way forward."
  - "Cards 8, 9 and 10 prescribe Stuck and refusal branches in their `**Intent:**`, among them seam read errors, an empty parent branch and a driver-busy resume refusal during the automatic Publish retry. Settle whether each prescribed branch has a named input and a named test that reaches it."
  - "Cards 1 and 2 both plan tests of the push retry loop, `gitrepo.PushWithRetry` in card 1 and `TestPushBranchWithRetry` in card 2, whose wrapper hands its loop to `PushWithRetry`. Settle whether each planned row covers behavior no other planned test covers."
  - "Card 11 edits the orch stencil's `## After landing` and Escalations text and the `orch-template-role.md` claims in `contracts/stencils`. Settle whether the stencil's run-loop wording on halted children, `internal/orchcli/doc.go`'s run-loop step 5 and the claims target agree with the automatic Publish retry and the Land row."
---

Focus entries restate the sites they depend on, and none caps severity.
