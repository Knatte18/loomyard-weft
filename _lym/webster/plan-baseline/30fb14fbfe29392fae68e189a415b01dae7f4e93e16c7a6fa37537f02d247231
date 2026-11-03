# Card 3 — runs-strand-beside-status

**Create:**
- `plan:internal/orchcli#RunsStrandOrder` -> `const RunsStrandOrder = 0`

**Edit:**
- `internal/orchcli#RunsStrandName`
- `internal/orchcli#runsStrandAddSpec`
- `internal/orchcli#orchCLI.ensureRunsStrand`
- `internal/orchcli/doc.go`
- `internal/orchengine/doc.go`
- `docs/overview.md`
- `internal/loomengine#LoomStatusStrandName`
- `internal/orchcli#TestStart_EnsuresTheRunsStrandBeforeTheOrch`

**Uses:**
- `internal/orchcli#newStartHarness`
- `internal/orchcli#fakeStrands`

**Intent:**
D3's runs half and D4: the runs strand takes the first overview place, beside a loom status strand and never in its place.

- `RunsStrandOrder` is the runs strand's overview order key, zero, so the beam overview is the session's first window; it is exported because loomcli derives the status strand's key from it (card 4), and it is the one declaration of the value (`PATTERN-named-constants`).
- `runsStrandAddSpec` sets `Display.Order: RunsStrandOrder` beside the anchor and the title; its doc comment calls the runs strand's window the session's first overview window.
- `runs.go`'s file header calls the runs strand's window the session's first overview window.
- `ensureRunsStrand` stops looking for a loom status strand: with no runs strand it adds one; a current runs strand is kept and a stale one replaced, as today.
  The `loomStatus` lookup, its replace case, its warning text and the `loomengine` import go; the doc comment drops the two sentences about replacing a loom status strand.
- `RunsStrandName`'s comment drops "A loom bootstrap in a beam's session reads it to leave the session's one overview window to this strand"; it is the runs strand's role.
- `LoomStatusStrandName`'s comment drops "and a beam's orch bring-up reads it to take the session's overview window from that strand".
- `doc.go`: the runs-strand sentence says the runs strand holds the session's first overview window, beside a `loom-status` strand a pawl run's `lym loom start` added, and keeps the warning half.
- `internal/orchengine/doc.go`: the bring-up sentence calls the runs strand "the session's first overview window"; its "the beam's window is the first window" half stays.
- `docs/overview.md`: the batten entry's sentence "Ensuring it replaces a `loom-status` strand a pawl run's `lym loom start` put in the overview window first" says ensuring it adds the runs strand beside such a strand, as the session's first overview window.
  The orch entry's clause "ensuring it replaces a `loom-status` strand a pawl run's `lym loom start` put in the overview window first" says the same.

Tests, `TestStart_EnsuresTheRunsStrandBeforeTheOrch` (tier0):
- `wantDisplay` gains `Order: RunsStrandOrder`.
- The row `a loom status strand holding the overview is replaced by the runs strand` becomes `a loom status strand gets the runs strand added beside it, untouched`: strands `{s1 loom-status live}`, `wantCalls` opening with `add`, sidecar `added-runs`; the row asserts the fake recorded no replace and no removal, so the status strand, its pane and its window are left alone.
- The two stale-runs rows beside a loom status strand keep their replace of `r1`.

D5 bound: a pawl driver an older build started runs the old `ensureRunsStrand` in-process until it ends; the record accepts that, and the next `lym beam start` under this build adds a dropped runs strand back.

**ImpactSummary:** `ensureRunsStrand` has one caller, `beamUp`, whose report and warnings keep their shape bar the dropped loom-status warning; `LoomStatusStrandName` keeps its value and its loomcli users, only its comment changes.

**Commit:** 3: orchcli: add the runs strand beside a loom status strand, first in overview order
