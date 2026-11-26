```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens review of the dobby run-watch plan: I traced each guard, branch and test row the cards add to a real entry point.

**1. MEDIUM — D1/D4, card 3 episode stamp.** A second identical block at the same row can leave the history length unchanged, so the second stop is missed.
- The card 3 row "a child blocked, then blocked again at the same row with a longer history" uses a state production often cannot reach.
- Evidence: a block appends a `Stuck` entry through `appendOrFold` (`internal/shedengine/reroute.go:49`, `:70`).
- `appendOrFold` folds a budget-exempt `Stuck` into the previous one when the producer, output, routed-to and gate attempts all match (`internal/shedengine/run.go:706`, `foldsInto`). It only increments `Repeats` and sets `LastAt`, and `len(History)` stays the same.
- Result: block, resume and block again with no tick between gives an equal `Episode`. Under `stops` the reading equals `Last` and no line goes, against the acceptance criterion "posts a second line, through the episode stamp".
- The same folding applies to batten's own `HistoryLen` stamp (card 2) for a repeated batten-level stuck.
- Fix: make the stamp the history length plus the last entry's `Repeats`, or its `LastAt`. `ChildReading` and `RunView` would then carry that value. This narrows D1's wording, so it goes to the parent as a recorded decision.

**2. LOW — D6, card 6 holder tree.** For a gate run in-process, the holder's tree can be the whole driver's process tree, and the notice line has no cap.
- Evidence: `internal/gatecli/rungate.go:32` calls `runTest` inside the calling process. That process is the long-lived Go-side gate caller, so `os.Getpid()` is that caller. Its `Holder.PID`, and `runMarkerEntry(os.Getpid())` in card 5, name it.
- `Tree(pid)` in card 6 then lists every descendant of that process. `gateHoldLine` renders one `; child pid …` clause per descendant, with no bound.
- The guard can be reached. The site it reports on is wider than "the holder's tree" suggests, and the card neither bounds nor notes it.

**Outside my lens, in one sentence:** card 5's row expects a stand-in that "waits on a release file", but the rig's stand-in holds until it is killed (`internal/gatecli/detachfront_integration_test.go:40`, `holdEnv`), and the card lists no change to the stand-in.

Calls made:
- Grepped shedengine history appends, `runfsm` landing, configengine type checks and the gatecli detach rig: my context lacked how history grows, how derivation works and the rig's shape.
- Grepped shedengine outcomes and append sites, and gatecli's `runTest` and `acquireSlot` callers: my context lacked whether a block appends history and which process holds the slot.
- Read `appendOrFold` and `foldsInto` at `internal/shedengine/run.go:700-730`: my context lacked the fold rule.
- Grepped configengine type checks and lymcwd `NamedWorktree` and `WorktreePath`: my context lacked whether a float key is type-checked and whether the holder path and the pair path match. Neither turned out to be a defect.
