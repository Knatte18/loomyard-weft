---
round: 1
focus:
  - "internal/state/state.go: ReadJSON skips the lock only on a not-exist error from opening the lock file, any other lock error still fails the read, and read failures wrap ErrRead and decode failures ErrDecode with %w; settle whether the unlocked fallback is bounded as the decision record says."
  - "Every former ReadJSONNoMkdir caller and every caller that relied on ReadJSON's MkdirAll: settle whether any writer or lock taker now fails because its directory was never created (raw os writes, lock files beside the data); also that no reference to ReadJSONNoMkdir remains."
  - "internal/fabricengine ReadOrigin and its new staging and junction predicates: settle whether ReadOrigin creates no .weft directory and seeds no exclude for a read, WriteOrigin keeps the creating path, and orchcli names no fabric path and runs no git."
  - "internal/orchcli/live.go HubOrchs: settle whether it skips a worktree under a .fabric-wt-staging-* directory before resolving it, skips a pair whose _lym is not a link before reading its origin, logs no warning for it, and still resolves through lymcwd; check the junction test goes through internal/fslink."
  - "Topology.Add ordering: settle whether the junction wiring now runs right after the weft worktree is created or adopted and moved to its anchor, ahead of the local-branch marks and writeLaunchers, on both the create and the adopt path, with no failure-path cleanup regressed."
  - "internal/websterengine/state.go LoadState: settle whether it creates nothing for a worktree with no scratch directory and returns no state."
  - "Tests against the acceptance criteria: settle whether the fabricengine test holds Add between weft creation and wiring and asserts no <warp>/_lym and no .weft under staging, whether each criterion has coverage, and whether any new test duplicates an existing one (test economy)."
  - "internal/state/doc.go, internal/orchcli/doc.go and PATTERN.md: settle whether the docs land in the same diff for the changed read behavior and the new HubOrchs skips, and whether any claim in _lym/landing/summary.md is not borne out by the diff."
---

Seed focus from the board entry and decision record; the wide mechanical rename of `ReadJSONNoMkdir` callers is covered by the verify gate and is not a focus on its own.
