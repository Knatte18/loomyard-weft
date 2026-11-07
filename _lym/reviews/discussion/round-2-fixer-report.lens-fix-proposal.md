```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out (F1–F5), and build concrete cases where a proposal is wrong or breaks something elsewhere in the record.

Fixed as proposed: F1, F2, F3, F4, F5. I found no case against F2, F4 or F5. F1 and F3 have one finding each.

**1. MEDIUM. F1 does not say which branch of the records bare to compare against (D2 last bullet, and the D2 criterion "leaves the board's HEAD unequal to the records bare's branch tip").**
- **The case:** the records bare holds more than one branch. It holds the board's Bolt branch: `publishBoard` pushes it and `originBoard` commits to `branch` in `hub.RecordsBare`, at `originreconcileended_integration_test.go:63` and `:70`. It also holds the weft records branches that `PrimeRecords` and `PairRecordsSibling` commit to, at `ensure_integration_test.go:396-399`.
- **How it breaks:** a builder reading "the records bare's own branch tip" as the bare's HEAD branch would compare the board's HEAD with a weft tip. Every build would then fail naming the key.
- **What the rule means:** the board's HEAD against the bare's tip of the board's current branch. The text should name that branch.

**2. LOW. F3's rule covers seams a test installs, but not once-only registrations a test makes before copying (D2 Bound: "a build calls no package-global seam setter or once-only registration, and a test that installs a process-global seam takes its copies before installing it").**
- **The case:** `TestReconcileOrigin_Ended` calls `boardkit.InstallReplay(t)` before any copy (`originreconcileended_integration_test.go:107`). Under D7, `endedHub` becomes a template that this test may be first to build.
- **How it breaks:** suppose a build reaches a Bolt replay. Then it passes when this test builds first. It fails with `ErrReplayerMissing` when an unregistered test builds first (`boltreplay.go:164`). Because D2 marks a failed template failed for every later caller, the outcome depends on test order.
- **The gap:** the Bound says a build runs under registrations already installed, but neither the rule nor the D7 review criterion catches a build whose result depends on one.
- **Not verified:** I did not confirm that `endedHub`'s steps (`originPair`, `writeStatus`, `publishBoard`) reach a replay.

Further tool calls:
- Read `internal/hubreconcile/ensure_integration_test.go:385-519`, to check whether `newStaleHub` depends on the seam installed before it (F3's reorder rule). My context held only grep lines for it.
- Grepped the non-test files of `internal/fabricengine` for `RegisterBoltReplayer` and `boltReplayer`, to see whether a missing registration fails loudly or silently changes a build. My context lacked `boltreplay.go`.
