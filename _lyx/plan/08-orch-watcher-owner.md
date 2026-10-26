# Card 8 — orch-watcher-owner

**Create:**
- `internal/orchengine#OwnerWatch`
- `internal/orchengine#Watcher.WatchOwner`
- `internal/orchengine#TestWatcher_StopsTheOrchOfABeamOwnedElsewhere`
- `internal/orchcli#beamOwnerElsewhere`

**Edit:**
- `internal/orchengine#Watcher`
- `internal/orchengine#Watcher.tick`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/orchengine/doc.go`
- `internal/orchcli/doc.go`

**Intent:**
D7: a beam's two parts leave a machine that no longer owns it; the dobby already exits on its owner read, and now the orch's watcher stops the orch strand too.

- `OwnerWatch` holds `Read func() (owner string, elsewhere bool, err error)`, the beam's owner from the local Bolt, and `Stop func(guid string) error`, which removes the orch strand.
  `WatchOwner` tells the watcher both; a watcher never told one reads no owner.
- `Watcher.tick`, after the strand-alive check, reads the owner: a read naming another machine calls `Stop` on the recorded strand, records `WatcherExit` as `owned by machine <owner>`, logs the owner at Info and ends the watcher as done; a failed `Stop` is the tick's error, retried on the next tick.
  A failed read reads as not owned elsewhere and is logged once per episode through a `logger.Episode` on the watcher, recovered at the next good read, as the dobby's own owner read is; the watcher then goes on with its cycle.
- `orchCLI.watchCmd` wires it: `beamOwnerElsewhere` reads the owners from the hub's local board through `boardengine.ReadOwners` and this machine's name from `machinename`, the prime's owner `Owners.Prime` and a beam's `Owners.Effective(slug, hubgeom.ParentOf)`, and answers whether it names a machine other than this one; `Stop` is the receiver's `strands.RemoveStrand`.

`TestWatcher_StopsTheOrchOfABeamOwnedElsewhere` drives `Tick` over the fake session with an owner watch: a read naming another machine stops the recorded strand once, records the exit and returns done; a read naming this machine or none goes on; a failing read goes on with no stop; a failing stop returns the error and keeps the watcher running.
The docs record the check and its bound: the verdict is as fresh as this machine's local Bolt, and a watcher started by an older build reaches it only once it restarts, `lyx beam status` showing the orch up meanwhile.

**ImpactSummary:** one new optional watcher seam, read only by a watcher that is told it; the orch module's watch verb is its one production caller.

**Commit:** 8: An orch's watcher stops the orch of a beam another machine owns
