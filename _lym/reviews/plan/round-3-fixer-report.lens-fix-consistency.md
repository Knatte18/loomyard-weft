```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set's agreement with sibling cards, decisions and rows, and find any changed text whose twin elsewhere was left as it was.

- **MEDIUM**: `06-pusher-target-kinds.md`, the `PusherSync` sentence ("pushes the Bolt once through `Bolt.Push` with `OwedUnrecorded` and the single transport attempt card 5 gives the pusher's pushes"). The step relies on a mechanism the twin card does not provide, so it cannot be carried out as written. Card 5 puts `SingleAttempt` on `fabriclayout.PairPush` only, and `PushPairAnchored` is the only thing that passes it on (`05-pusher-interval.md:63`). `Bolt.Push` takes only `SyncOptions` (`internal/fabricengine/bolt.go:121`), and `SyncOptions` has no single-attempt field (`internal/fabricengine/fabric.go:74-95`). To make it workable, card 5 would need to add the option to `SyncOptions` itself. Card 10's `PushPrimeCodeTip` ("no transport backoff of its own") already relies on the same thing.
- **MEDIUM**: `11-parent-pusher.md` step 1 ("clear the parent's refused-push record unless this pass held a landing back") against step 2 ("drop it … and clear the parent's refused-push record"). Step 1 now guards its clear, but step 2's clear, which runs later in the same pass, is still unconditional. A step-1 `held_back` record for a run owned elsewhere is therefore wiped by a drop of a different landing in that pass. `SettleRefused` runs both steps too, so it has the same gap. Step 2's clear needs the same guard.
- **LOW**: the `ImpactSummary` of `12-finalize-local-landing.md` is now stale. It still reads "Removes Finalize's synchronous pushes and four seams … and adds a trailer to the landing commit". The fix also adds `MergeOptions.NoFastForward` and `gitrepo.Repo.MergeStartNoFastForward`, and changes `Fabric.Merge` for every non-squash caller that sets the field.
- **LOW**: the `ImpactSummary` of `06-pusher-target-kinds.md` ("Adds two target kinds, three verb flags and one refusal read") leaves out the new `Board.PusherSync` method in `boardengine`. It also leaves out the change to the board's `Ahead`, which now reads uncommitted files.

Calls made:
1. Grepped the plan cards for `HasUnpushed`, `Board.Sync`, `PullBolt`, `SettleRefused`, `SingleAttempt`, fast-forward and `BackoffSleep`. I needed the sibling cards' wording, and my context holds only the cards the fixer read.
2. Grepped `internal/gitrepo` for its Client Boundary pinned list and for tests that pin the `--ff --no-commit` form. I needed to know whether a new merge form must be listed there; nothing in my context showed it.
3. Read the non-squash step of `internal/landingshed/finalize_integration_test.go`. I needed to know whether an existing test assumes a fast-forward non-squash landing; it asserts no parent count.
4. Read `internal/gitrepo/merge.go:318-332` and the boundary docs. I wanted to confirm the pinned list is only a doc mention with no test behind it, which held, so I raised nothing there.
