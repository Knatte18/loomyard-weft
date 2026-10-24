```yaml
lens: fix-claims
outcome: reported
```

Checked every claim the round-1 fix added or rewrote against the code at HEAD.

Claims list (✓ holds, ✗ see the findings below):
- ✓ D5: the Stuck for a dirty parent names "commit or stash them in the parent pair" and a resume (`internal/landingshed/finalize.go:406`, `:419`).
- ✓ D7: an adopted pair keeps the origin record already on the beam's weft branch (`internal/fabricengine/add.go:157`, `internal/fabricengine/doc.go:596`).
- ✓ D11: `lyx orch status` reports whether the strand is live and `watcher_live` (`internal/orchcli/status.go:52`, `orchengine.WatcherLive`).
- ✓ D13: the notifies from hibernate and fabric's push notifier act on no run (`internal/hibernatecli/deps.go:112`, `internal/orchcli/pushnotifier.go:46`). `ResolveParent`'s `.Worktree` is the recorded `ParentWorktree` (`internal/hubgeom/parent.go`).
- ✓ D10: cancel's own order is to end processes and strands, then clear the claim and the records. Its Long text says "with origin unreachable it only ends the run's processes and strands", which is the split promote uses.
- ✓ D14: `lyx board promote` turns a note into a task (`internal/boardcli/cli.go:151`).
- ✓ D8: the only run whose agents work in a beam is the beam's own landing. The `beam` recipe's darn Finalize fixer works in the beam pair, and every other run works in its own pair.
- ✗ D4: a spawned entry's `parent_beam` is the parent its pair lands into.
- ✗ D8: a `--window` window in a beam's session is safe because of D11's refusal.
- ✗ D11: `lyx orch stop` is a way forward that clears the orch refusal.

Findings:

1. MEDIUM. D4, the new `parent_beam` sentence ("so a spawned entry's `parent_beam` is the parent its pair lands into, also for a member spawned under a grouping project that later became a beam").
   The claim holds only on the machine that holds the pair.
   Evidence: `ParentOf` reads the origin record through `fabricengine.ReadOriginFor` over the prime's location (`internal/pairteardown/originseams.go:39-44`), which is the local pair.
   On a machine without that pair, D4 falls back to `ParentBeam`, which gives the beam, not the prime.
   So the D8 stencil filter on that machine's orch still misroutes the entry, and the D4 acceptance line holds only on the owner machine.

2. MEDIUM. D8, the new `--window` sentence ("that session is the prime's or a beam's, and no landing of a member tears a beam down while a member pair exists (D11's refusal)").
   The bound does not cover a window the beam's session holds for a run that is not one of its members.
   Evidence: D8 now lets a session in a beam act on any run of the hub. So `lyx batten run B --window` from the beam can open the window of a run B whose parent is the prime inside the beam's reed session.
   D11 refuses only while an open entry or a local pair names the beam, so the beam can land while B's window lives in its session.
   The beam's Worktree-Teardown then ends that session, and B's window with it. This is the same failure F1 described, moved from a task pair to the beam.

3. LOW. D11, the new orch refusal ("naming `lyx orch stop` in the beam's worktree, so a landing leaves no orch strand or watcher behind").
   `lyx orch stop` removes the strand but does not stop the watcher, so a land run right after the stop can still be refused.
   Evidence: `internal/orchcli/stop.go:1` says stop removes the strand "and let[s] the watcher notice and exit on its own".
   A watcher still live after the stop meets the same refusal again, and the way forward does not say to wait for the watcher to exit (`orchengine.WaitWatcherGone` exists).

Tool calls:
- Read `internal/adoptcli/doc.go` to check what adopt does with a pair's origin record and ownership; my context held only one line of it.
- Grepped `internal/fabricengine` for how `Add` keeps an existing origin record, for the D7 beam-mark claim; my context did not show it.
- Read `boardcli`'s promote help, `orchengine.WatcherLive` and batten's start guard, for the D14 and D11 claims; my context did not show them.
- Grepped for `ParentOf` to see where it reads origin records, for the D4 claim; my context had only the support log's mention of it.
