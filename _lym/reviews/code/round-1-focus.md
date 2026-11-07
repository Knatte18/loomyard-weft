---
round: 1
focus:
  - "internal/fabricengine/post-checkout.sh: settle whether `${WORKTREE_ROOT%/*}` and `${WORKTREE_ROOT##*/}` agree with dirname and basename for every path `git rev-parse --show-toplevel` can answer, including a root-level worktree such as `/x`, where `%/*` yields an empty parent."
  - "internal/fabricengine/hook.go `chainedHookPreamble`: settle whether the `case \"$0\"` expansion yields the right SCRIPT_DIR for a `$0` with a slash (including `/hooks`-style root paths) and for one without, and whether the Go raw-string constant and the idempotency check on the chained content still hold."
  - "internal/fabricengine/hook_test.go: settle whether the new checkout probe in `TestInstallPostCheckoutHook_ChainIdempotent` covers the acceptance criterion that the chain wrapper runs `post-checkout.user` when executable, and whether the warn-on-mismatch and silent-on-match behavior of the hook is still covered; check the probe is not redundant with an existing test."
  - "Scope against the decision record D5: `newRecordsFixture` and the `EndedSet` rows of `TestRecordsReplay` in replay_integration_test.go, and every `hubforge.NewHub` site of internal/adoptcli, take `hubforge.CopyHub`; settle whether the `EndedSet` rows reach `CopyHub` through `newRecordsFixture` or still build their own hub, and whether any adoptcli site was missed."
  - "Acceptance criterion `Review:` each kept `hubforge.NewHub` site names the clone or wiring behavior that needs it: settle whether the diff leaves any `NewHub` site in internal/adoptcli or in the named fabricengine fixtures without such a reason, and that no site outside D5's list (the internal/hubreconcile sites) was touched."
  - "`hubforge.CopyHub` with `Shape{Anchor: \".\"}` against `NewHub(t, \".\")`: settle whether the copy carries the same hub state the converted tests rely on (the weft primary on its suffixed branch, the records bare, AddPair and PrimeWorktree), and whether parallel tests sharing one template stay isolated."
---

Diff is small: two commits, seven files. The shell and wrapper expansions carry the correctness risk; the fixture swaps carry the scope risk.
