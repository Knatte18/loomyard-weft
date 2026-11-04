# A landing notice survives a late request, a dobby outage, an absent orch

`landing-watch` has dobby send the orch one creel line when a task with `notify_on_land` lands.
Three known limits left that line lost: a request set after the run passed Land, a landing while dobby was down, and an orch that is not live to take the post.
This change closes all three.

## What changes

- **Late request.**
  `lym board upsert` with `notify_on_land: true` on a done entry now prints a notice that the run already landed and that its parent's dobby sends the line at its next tick.
  A payload that only asks for a notice (slug and `notify_on_land`) on a slug the board no longer holds is refused, since Land already pruned it and the upsert would file a new entry for a landed run.
- **Landing missed while dobby was down.**
  `lym board prune` and Land's prune keep a done entry that still carries `notify_on_land`.
  Dobby reads the board's done state as well as its own state record: a done entry of its worktree, owned by this machine, that carries the request and has no memory gets its landed line at the first tick that reads it, naming the commit when its run is listed past Land.
  Dobby clears the field once the line went out, which lets the next prune drop the entry.
- **Absent orch.**
  The three-attempt cap is gone.
  Dobby keeps an owed line and an owed clear in its state record and retries at every tick until they succeed, across a restart and a re-exec; each failing post is logged once per episode.
  Creel now accepts mail to the fixed name of an orch of this hub (`<shortname>:orch`, `<shortname>:<slug>:orch`) whether or not a strand of that name is listed, and holds it unread until the strand is back.
  The sweep no longer returns unread program mail for such an orch to a sender that has no mailbox.

## Design decisions

- A done board entry is read as landed whatever set its status, so an entry marked done by hand with `notify_on_land` gets a landed line for a landing it never made.
  This is documented in `dobbyengine/doc.go`.
- Under `land_notice: off` the line is skipped, but the request is still cleared, so prune does not keep the entry forever.
- A beam entry's `Parent` and owner are now read for any entry carrying `notify_on_land`, not only for a released task.
  When they do not read, a released task still fails the whole read; a flagged entry only waits, logged once per episode, so it never blocks other starts.
- A run first seen past its Land row whose entry asks nothing still gets no line, so `land_notice: all` does not notice a landing dobby did not see ahead of Land.
- The orch-role contract, `docs/overview.md`, the board skill and the `prune` help describe the kept entry.

## End to end

The notice is shown end to end in a test, not by hand against a live orch.
`internal/dobbyengine/landnotice_creel_test.go` runs dobby's notice against a real creel store, scheduler and sweep, with the orch absent and then back.
No live-orch run by hand was made.

## To check by hand

- `internal/dobbyengine/landnotice.go`, `takeDoneEntries` and `internal/battencli/dobbyreads.go`: a done entry's notice honours the owner machine and parent worktree; a flagged entry whose parent or owner is unreadable waits without failing the read.
- `internal/dobbyengine/landnotice.go`, `rememberRuns` and `settleLandNotices`: the rows for a failed clear, an unreadable `land_notice` mode and a restart with an owed line each keep the memory and retry at the next tick.
- `internal/dobbyengine/landnotice_creel_test.go`: the test stands in for a live orch; run the notice once against a live orch if that is wanted.
- `internal/creelengine/sweep.go`, `waitsForHubOrch`, and `internal/creelengine/pool.go`, `isHubOrchName`: unread program mail for an unlisted hub orch stays under the orch's name and is never returned, and a mistyped orch slug is refused only when it is no hub orch name.
