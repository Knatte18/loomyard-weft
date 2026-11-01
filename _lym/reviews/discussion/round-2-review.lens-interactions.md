```yaml
lens: interactions
outcome: reported
```

Interactions lens: I checked every pair of decisions in the record that act on the same state, then each acceptance criterion and constraint against the decisions it touches.

**Pairs checked**
- D1 × D2 (store types and the write path over `heddle.json`): held.
- D1 × D7, D1 × D8 (a replay rewrite keeps unknown keys through D1's types): held.
- D1 × D3 (`order` and `targets` against the target table): held.
- D2 × D7 (one detached sync commit can carry a board change and a heddle change, which both replayers apply): held.
- D2 × D6 (a write never renders into a live worktree, and a `heddle.json` change alone fires nothing): held.
- D2 × D7, locking (heddle writes and `replayBolt` both run under `BoardWriteLockFile`): held.
- D3 × D5 (a write off the exclude list is refused, removing a stale render is not, and lines are never unseeded): held.
- D3 × D6, ordering (D5's lines are seeded before either event, and "made" fires last): held.
- D3 × D6, re-render paths (a take-in of `AGENTS.md`, `CLAUDE.md` or `.claude/CLAUDE.md` re-renders only the affected target): held.
- D3 × D6, origin starts tracking `AGENTS.override.md` (git overwrites the ignored render, then the re-render skips the file without a marker): held, and Open risk 1 already names it.
- D3 render lock × D6 seam failure (a lock failure becomes a warning): held.
- D4 × D1 (`claude-prompt` and `codex-prompt` are targets with no file): held.
- D4 adopted re-read × D6 "never pushed into a live worktree": held, since D4 states its own bound and Auto-mode assumptions records it.
- D5 × D6 (`lym fabric reconcile` seeds lines but fires no "made"): held, and Open risk 3 covers it.
- **D6 × D7, the detached Bolt sync: broken (F1).**
- D7 × D8 (a heddle refusal is tagged `set-aside/replayer/heddle/...`, and a board loss in the same commit gets its own tag): held.
- D7 run ledger × weft replay's ended set (`weftEndedByOrigin` reads through the ledger): held.
- D7 × D9 (claimed paths are Bolt-root-relative while the board sits at Bolt's root): held.
- D9 × D2 and D9 × Scope-Out (the write lock's place comes from `BoltDirName`, while its file name `board.lock` stays): held.
- D9 × Constraint PATTERN-cwd-resolution (`lymcwd`'s import set is unchanged): held.
- **D9 × Constraint PATTERN-leaf-packages × `## PATTERN changes`: broken (F3).**
- **D9 × its own D9 acceptance criterion: broken (F2).**
- D4 × Constraint PATTERN-shuttle-provider-seam: held.
- D2 × Constraint spawn-observability and run-process-owner: held.
- D3 × Constraint durable-vs-ephemeral (the render lock sits under `.lym`, the rendered files outside it): held.
- The D1, D2, D3, D4, D5, D7 and D8 acceptance criteria against their decisions: held.

**Findings**

**F1 — MEDIUM, consistency.** Location: D6, its paragraph on detached processes, and the D6 acceptance criterion.
- Summary: D6 names the board's Bolt sync as a detached process that reaches the take-in event, and an acceptance case asks for "a failing seam in a detached Bolt sync". The Bolt sync never moves a checked-out code branch, and D6 itself says a records or Bolt branch fires nothing. That case cannot be met.
- Evidence: `internal/boardengine/sync.go:36-58` (`Sync` only commits and calls `bolt.Push`); `internal/fabricengine/bolt.go:122-128` (`PushRecorded` → `pushClassified` with `retryBoltPush`, Bolt side only); `internal/fabricengine/pushclassified.go:677` (`retryBoltPush` replays Bolt only).
- The detached processes that do move warp are the ones calling `ReconcileOrigin` or `FastForwardPair`, such as `battencli` and `hibernatecli/reconcile.go`.
- Fix: name one of those as the example, and retarget the acceptance case to it.

**F2 — LOW, consistency.** Location: D9, the third and fourth bullets against the D9 acceptance criterion.
- Summary: The criterion allows the `_board` literal in only "one Go declaration and in `TestEnforcement_GeometryLiterals`'s policed-token list". The same test also spells the literal as its owner-row map key and in its planted fixture sources. D9's bullet narrows that owner row but never says the key and the fixtures take the name from the constant.
- Evidence: `internal/lymcwd/enforcement_test.go:209` (policed list), `:225` (owner row keyed `"_board"`), `:412-413` (fixture sources).
- Fix: either say the owner-row key and the fixtures take the name from `lymcwd`'s constant, or widen the criterion's exception to cover them.

**F3 — LOW, consistency.** Location: D9's first bullet, the PATTERN-leaf-packages Constraint and `## PATTERN changes`.
- Summary: D9 says, in the present tense, that PATTERN-leaf-packages's background "admits" `runproc` → `lymcwd`. It does not: the `runproc` section lists only the standard library, `proc`, `state` and `logger`. The Constraint says "see `## PATTERN changes`", but that bullet carries the overview text unchanged and does not mention `runproc`. So no line of the record states the background edit as a deliverable.
- Evidence: the leaf-packages background, `## internal/runproc` (it lists those four imports only); D9's first bullet; `decision-record.md` `## PATTERN changes`, the leaf-packages bullet.
- Fix: state in D9, or in the Constraint, that the background's `runproc` section gains `internal/lymcwd` for the Bolt directory name.

**Tool calls**
1. Read `fabricengine/bolt.go` `Push`, `PushRecorded` and `Sync`, and the function list of `pushclassified.go`. My context did not show whether the Bolt sync's push path moves a warp branch.
2. Grepped for callers of `ReconcileOrigin`, `FastForwardPair` and `Pull` outside fabric. My context lacked which detached processes move a warp code branch.
