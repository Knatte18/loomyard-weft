# Card 7 — Creel never retypes a pasted send and skips a torn-down worktree

**Edit:**
- `internal/creelcli#mapDeliveryError`
- `internal/creelcli#hubDelivery.Strands`
- `internal/creelcli#hubDelivery.worktreeStrands`
- `internal/creelcli#stubOps`
- `internal/creelcli#stubOps.Status`
- `internal/creelcli#newStubbedDelivery`
- `internal/creelcli#TestHubDelivery`
- `internal/creelcli/doc.go`

**Create:**
- `plan:internal/creelcli#worktreeTornDown` -> `func worktreeTornDown(path string) bool`
- `plan:internal/creelcli#errWorktreeGone` -> `var errWorktreeGone = errors.New("creel: the worktree is gone")`
- `plan:internal/creelcli#TestWorktreeTornDown` -> `func TestWorktreeTornDown(t *testing.T)`
- `plan:internal/creelcli#TestHubDelivery_SkipsAWorktreeThatNoLongerResolves` -> `func TestHubDelivery_SkipsAWorktreeThatNoLongerResolves(t *testing.T)`

**Uses:**
- `internal/creelcli#seedReedState`

**Intent:**
D7, creel's part of issue #828.

`mapDeliveryError` maps `shuttleengine.ErrSentTextInDraft` and `shuttleengine.ErrForeignBoxText` onto `creelengine.ErrNotLanded`, in a case checked before the generic `ErrSessionBusy` case, since both wrap `ErrSessionBusy` and are raised after the paste: creel's contract is that `ErrBusy` means nothing was typed, so the delivery worker journals such an inject unconfirmed instead of requeueing and typing it again.
Only a hold before the paste (`ErrSessionBusy` alone, or `ErrPaused`) stays `ErrBusy` and is requeued.
Bound: an inject left beside a draft is not retyped; shuttle's `pending-send.json` resolves an `ErrSentTextInDraft` one at the next send, and an `ErrForeignBoxText` or recordless one is lost rather than typed twice.

`Strands` skips a listed worktree that no longer resolves as a worktree, instead of failing the whole listing over a worktree torn down mid-lookup.
`worktreeStrands` returns `errWorktreeGone` (wrapping the cause) in two cases, and `Strands` skips such a worktree and goes on.
The second case's check is one helper the state read, the wire and the status step all call on their failure, so the rows below that reach it through the wire and the status step pin it for all three:
- `lyxcwd.ResolveWorktree` fails with `lyxcwd.ErrNotAGitRepo`, which also covers a gone directory; a not-a-git-repository resolve cannot tell a teardown from a broken `.git` link, so any such worktree is skipped whatever the cause, and its strands get no delivery until it resolves again;
- the reed state read, the wire, or the reed status step fails (any error but `reedengine.ErrNoSession`, which still reads as dormant) and the worktree's directory, checked with `os.Stat` after that failure, no longer exists, since a teardown can land after a successful resolve.
Any other resolve failure, and a state, wire or status failure while the directory exists, still fails the call naming the worktree, so an unreadable strand is never mistaken for an absent one.
`worktreeTornDown` reports a teardown's mark: the worktree directory is gone, or its `.git` is a file whose `gitdir:` line names a directory that is gone.
The skip logs `creel: skipped a worktree that no longer resolves` at Info when `worktreeTornDown` answers true and at Warn when it answers false, with the path and the cause, so a cause that is no teardown shows.
The Warn line also names the way forward that ends the skip: `git worktree repair` from the hub's prime, since a worktree whose `.git` link is broken but whose directory stands resolves again once the link is repaired.
`Strands`' doc comment states the skip and what still fails.

`stubOps` gains an `onStatus func()` hook that `stubOps.Status` runs first when set, so a test can remove the worktree's directory between the resolve and the status step, and `onWire func()` and `wireErr error`, which `newStubbedDelivery`'s wire closure runs and returns for that worktree's stub, so a test can remove the directory and fail the wire.

`internal/creelcli/doc.go` gains the mapping of the two post-paste errors and the worktree skip with its log levels.

`TestHubDelivery`:
- the "Send and Interrupt map shuttle's sentinels" table gains rows for `ErrSentTextInDraft` and `ErrForeignBoxText`, each wrapped, mapping to `creelengine.ErrNotLanded` on `Send`, beside the kept `busy` row mapping to `ErrBusy`;
- "Strands fails naming a worktree that does not resolve" is removed from `TestHubDelivery`, which is `t.Parallel()`, and its case moves into the new top-level `TestHubDelivery_SkipsAWorktreeThatNoLongerResolves`, which is not parallel because `logcapture.CaptureVerbose` swaps the process-global logger and its rows assert an Info line:
  with the pair's `.git` replaced by an empty directory, `Strands` returns the prime's strands, skips the pair and logs the skip at Warn naming `git worktree repair`, since both the directory and its `.git` exist;
  a row seeds the pair's reed state, sets `onStatus` to remove the pair's directory and scripts `statusErr`: `Strands` skips the pair, returns the prime's strands and logs the skip at Info;
  a row sets the pair stub's `onWire` to remove the pair's directory and `wireErr` to fail the wire: `Strands` skips the pair and logs at Info;
  a row writes an unusable recorded anchor (`../escape`) into the board's anchor file, so the resolve fails with an error other than `lyxcwd.ErrNotAGitRepo`: `Strands` fails the call;
- a gone directory or gitdir never reaches the resolve through `fabricengine.List`, which marks such a worktree prunable and `Strands` already skips, so `worktreeTornDown`'s marks are pinned by a new table, `TestWorktreeTornDown`: a missing directory is torn down, a `.git` file whose gitdir is gone is torn down, a `.git` file whose gitdir exists is not, and a `.git` directory is not;
- "Strands reads a worktree without a session as dormant and fails on another status error" keeps its failing half, which now pins that a status error with the directory present still fails naming the worktree;
- "Strands fails naming the file on a corrupt reed state" is unchanged and still pins that a state read failure with the directory present fails the call.

**ImpactSummary:** One mapping case and one skip rule inside creelcli; the delivery worker's reading of `ErrNotLanded` and `ErrBusy` in creelengine is unchanged, and no other caller lists strands through this delivery.

**Commit:** 7: creel journals a post-paste failure unconfirmed and skips a torn-down worktree
