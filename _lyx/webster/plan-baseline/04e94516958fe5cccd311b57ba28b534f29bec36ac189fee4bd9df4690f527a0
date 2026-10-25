# Card 38 — orch-start-dobbies

**Edit:**
- `internal/orchcli#orchCLI`
- `internal/orchcli#Command`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli/doc.go`

**Create:**
- `internal/orchcli#TestStart_StartsTheDobbiesFirst`

**Uses:**
- `internal/dobbyengine#EnsureLive`
- `internal/hubgeom#DobbyStart`

**Intent:**
Make `lyx orch start`, with or without `--continue`, start every dobby this machine owns first, so one `lyx orch start` after a machine restart brings every run back (D2).
`orchCLI` gains a `startDobbies` seam, filled in `Command` with `dobbyengine.EnsureLive(hubgeom.DobbyStart(prime))` and nil in a hand-built harness; `startCmd` calls it before the beam-start refusal and the start lock, and a failure never stops the start: it becomes a Warn line on the envelope's `warning`, naming the failed worktrees and `lyx beam start` as the way forward.
The start's help names the dobby start, and `doc.go`'s start paragraph says so.
`TestStart_StartsTheDobbiesFirst` (tier0, the start harness) rows: a successful dobby start runs before the strand is launched; a failing one still launches the strand and the envelope's warning names `lyx beam start`; a `--continue` start calls the seam first too; a nil seam starts nothing.

**ImpactSummary:** One seam called first by the start verb, failing open; the rest of the start sequence is unchanged.
