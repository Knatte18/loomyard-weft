# Card 16 — slack-hook-operator-filter

**Create:**
- `plan:internal/slackcli#hookActionUnmarked` -> `const hookActionUnmarked = "unmarked"`
- `plan:internal/slackcli#operatorMarker` -> `const operatorMarker = "[operator]"`

**Edit:**
- `internal/slackcli#slackCLI`
- `internal/slackcli#slackCLI.replyHook`
- `internal/slackcli#wireProduction`
- `internal/slackcli/doc.go`
- `internal/slackcli#TestHook`
- `internal/slackcli#harness`
- `internal/slackcli#newHarness`

**Uses:**
- `plan:internal/hubgeom#Autonomy`
- `internal/slackcli#hookActionForward`

**Intent:** D8: while the orch's autonomy, read at call time, is true, the Slack reply hook forwards a turn's reply only when it opens with `[operator]`.

- `slackCLI` gains `orchAutonomous func() bool`; `wireProduction` fills it, from the location it is wired over, the hook's cwd resolved as for the other verbs,
  with `hubgeom.Autonomy(location, location.WorktreeName)`, the orch's own worktree, read on each call.
  A nil seam reads false, and so does a read that fails, which forwards every reply, as today.
- `replyHook`, once the presence reads slack and before it resolves the orch's client, returns `hookActionUnmarked` and posts nothing
  when `orchAutonomous()` reads true and the reply does not open with `operatorMarker`.
  With autonomy false it forwards every reply, as today.
- Bound: the filter drops only unmarked replies, only while the presence is slack and the orch is autonomous; the pane shows every reply,
  and the orch's direct posts of relays and notices to `<shortname>:SLACK` (card 17) are untouched.
- `doc.go` states the filter.

`harness` gains an `autonomous` field that `newHarness`'s wire seam copies into `orchAutonomous`.
`TestHook` (tier0) gains a field setting it and rows under a slack presence with the button on:
autonomy on with a reply opening `[operator] severe halt x:` forwards it;
autonomy on with an unmarked reply answers `unmarked` with no post;
autonomy off forwards both.

**ImpactSummary:** One early return in the reply hook behind one new seam; the prompt hook, the other hook events and the post path are unchanged.
