# Card 9 — loom status names a running verify

**Edit:**
- `internal/loomcli/reviewstatus.go`
- `internal/loomcli/reviewstatus_test.go`

**Uses:**
- `internal/verifytree/marker.go`

**Intent:**
D14: loom's `Hooks.Waiting` reports a running verify ahead of the parent-review note.
`reviewWaitingFor` composes a hook that reads `verifytree.ReadMarker` at `verifytree.NewPaths(...)`'s marker path under `verifytree.Dir(loc.AnchorPath())` and, when a live marker is present, returns its `Note()` (`verify <site> (attempt N, since HH:MM)`);
otherwise it returns the parent-review note as today.
A marker whose pid is dead is ignored, so a crashed verify never sticks in the status line.
`shedverbs`' `waitingNote` already serves both the one-shot status envelope and `--watch`, so both surfaces show the note with no change there.

Tests in `reviewstatus_test.go`: a live marker yields the verify note ahead of a pending review note;
a dead-pid marker yields the review note;
no marker and no review yields the empty note.

**ImpactSummary:** `lyx loom status` and its `--watch` tail gain the running-verify note; the review note keeps its text.
