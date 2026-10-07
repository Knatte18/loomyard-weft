---
round: 1
focus:
  - "internal/fabricengine/pushanchored.go: does the retry helper push a second time only when the fetch succeeded and HasUnpulled reported false, and does every other path return the bare gitrepo.ErrPushRejected from the first push, with no rebase, merge or force anywhere?"
  - "internal/fabricengine/pushanchored.go: do PushAnchored and PushPairAnchored take the same weft-side fabric.push.lock file CoalescePushBothAt takes, release it on every return path, and keep SkipGit and SkipPush returning before any lock is taken?"
  - "internal/fabricengine/pushanchored.go: does PushPairAnchored attempt the records side after a code-side failure, skip an unborn HEAD, wrap each side as 'push <side> side at <path>', and keep errors.Is(err, gitrepo.ErrPushRejected) true through errors.Join?"
  - "internal/lock/lock.go and internal/gitrepo/push.go: does AcquireWriteLockWithin map only a context expiry to (nil, false, nil) and wrap every other flock error, and does HasUnpulled return (true, nil) for no upstream and (false, err) for a spawn failure, as card 2 specifies?"
  - "internal/loomcli, internal/statuscommit, internal/battencli, internal/fabriccli: does any new or changed comment, error text or help string name a side outside fabricengine's error texts and fabriccli (PATTERN-fabric-vocabulary), and do the three PushAnchored callers pass StatusPushLockWait while batten's seam stays records-only?"
  - "internal/loomcli/commitrecords.go and contracts/specs/refusal-spec.md: do the two push-failure rows match each other in message text, class and way forward, with a rejection beside another failure taking the rejection row and neither row naming a command the agents' settings deny?"
  - "internal/fabriccli/weft_verbs.go and spawn.go: does `push` use PushPairAnchored with LockWaitUnbounded and drop the pull --rebase path, does `sync` hand the detached child both paths with one push_spawned entry per side, and do the bypass branch and Long/Short help match the code?"
  - "Per-card mechanical check and test economy: does each card's own test set match its Tests section, does any added test duplicate a covering test (naming it), and does any behavior the diff adds lack coverage (naming it)?"
  - "Doc comments in the diff: do they follow the Go comment conventions of the surrounding code and any rule in the repository's constraints document, with semantic line breaks and no width inferred from neighbours?"
---

Seed focus derived from the plan's five cards and the committed diff against `main`.
No earlier round exists; entries name where to look and the question to settle only.
