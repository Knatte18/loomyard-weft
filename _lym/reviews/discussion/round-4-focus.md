---
round: 4
focus:
  - "D6 in `_lym/discussion/decision-record.md` places the stalemate check after the soft-cap auto-accept branch of `settleUndecided` and before `softCapRule` in `internal/shedadapters/bouncer.go`; settle whether a window-open round whose ledger holds no design-wrong entry, with the auto-accept setting off, is accepted by `softCapRule` as today or escalated as a stalemate, and whether D6 and its acceptance criteria state which."
  - "D3's `carries` key bounds a carried finding's severity and location path; settle whether D3 also bounds its class, given that `GatingClass` in `internal/burlerengine/verdict.go` decides when the loop stops and a decision-class finding's class decides whether it goes to the fixer or the parent."
  - "D3 lets a reviewer mark a held decision-class finding `resolved` by quoting a listed title; settle whether the Decision states what that can let through and what bounds it, and whether the Go check named in the round-check entry enforces that bound."
  - "Where D1, D2 and D3 need to know whether round N-1 routed decisions, the fix row holds that fact (`NotifyParent`, `DecisionsEscalated`) and the review row does not; settle whether the record names where the fact is stored and who reads it."
  - "D3's last paragraph on `validate-review` says the verb skips worktree resolution and resolves the worktree from the file's path; settle whether the paragraph is consistent with PATTERN-cwd-resolution (`lymcwd` only), names where resolution starts given that `_lym` is a link out of the worktree, and names the inputs the builder needs."
---

Entries restate the sites they depend on.
The first two target the reopened keys; the rest target new round 3 design findings.
