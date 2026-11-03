```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code paths that already write, read or reset the state each decision touches. Three paths break a decision, five partly survive, and the rest hold.

**Findings**

1. `BLOCKING` — D1/D2, resume of an assisted review. The assistant's report path and ended-marker path sit in the half's fresh `os.MkdirTemp` instruction directory, so a probe cannot rebuild them, and resume leaves the chair waiting on files nothing writes.
   - Evidence: `Review` composes into `instructionDir("review")`, which is `.lym/burler/review-<random>`. `ProbeReview` and `ResumeReview` build the table from `reviewFilesIn(e.probeDirOf())`, which is `.lym/burler/round-probe`.
   - The seat probe matches a run by its exact output-file set, so a live assistant is never found. D1's resume rule then writes the ended marker at the probe-dir path.
   - The resumed chair keeps the prompt it started with, which names the original temp-dir report and marker. Neither is ever written, so the chair's `lym shuttle await` loop runs until the chair's own timeout. The live assistant is also left unstopped.
   - The fix needs a deterministic per-round report and marker path that the probe can rebuild, not one inside the temp instruction directory.

2. `BLOCKING` — D6's claim that an empty resume line means "reed cannot revive a dead Codex pane, which ends the run died" is false.
   - Evidence: `internal/reedengine/lifecycle.go` around line 866 uses `resumeCmd := ResumeCmd; if "" { resumeCmd = Cmd }`. `strand.go:621` has the same "stored-ResumeCmd-else-Cmd fallback".
   - So a reed resume after a reboot, or a revive, relaunches the full Codex launch line with its first-turn prompt in the same strand and run directory. That starts a second, fresh assistant review: Codex budget is spent again, and a new `SessionStart` lands in the old events file.
   - The record must either make the resume path inert for Codex (for example a resume line that only exits or marks the run), or state and bound the fresh relaunch.

3. `MEDIUM` — D3/D2: a list edit between spawn and probe changes the table's shape.
   - Evidence: `internal/shedadapters/burlerreview.go` records each round's fan names (`recordRoundFans`, `readRecordedFans`) so that a resumed reviewer is "probed and audited under the fan it was spawned with".
   - The record says nothing about recording the round's assistant model choice. If an operator edits `<segment>_assist_reviewer_model` mid-round, the probe builds a table with or without the `assist` seat, or on another engine, unlike the one spawned. The probe then misses seats, or finds a chair whose stencil (`burler-template-reviewer` vs `-assisted`) differs.
   - The assistant choice needs the same per-round record as the fans.

4. `MEDIUM` — D2: the copied `.assist-<seat>.md` is not covered by the review row's archive-on-exit.
   - Evidence: `burlerreview.go` `archiveRound` archives only `{ReviewPath, FixerReportPath, roundAcceptancePath}` through `archiveStaleOutputs`.
   - A failed attempt or round leaves its assistant copy beside the review path. A retry or later round whose assistant writes nothing keeps that stale copy, which is then misattributed to the new review.
   - The copy needs to join the archived set, or be removed before each attempt.

5. `MEDIUM` — D5: rendering the whole `config.toml` from lym's record erases the trust entry Codex writes for the main checkout.
   - Evidence: the hook-delivery report, case 5, shows Codex keys trust on the repository root. The prompt said "Trusting will apply to the repository root: …/loomyard-weft", so it wrote an entry for the main checkout, not for the linked worktree.
   - D5 adds the worktree root instead. If Codex does not honour a worktree-root entry for a linked worktree, the folder-trust prompt shows on every launch. The backstop accepts it and Codex writes the main-checkout entry, but the next Prepare's whole-file render drops that entry again. The result is a prompt per launch, with the "backstop" as the steady state.
   - The record should key the record's root as Codex does (the main checkout), or keep the entries Codex adds.

6. `MEDIUM` — D12: the `codex-file` target's re-render triggers miss `CLAUDE.md`.
   - Evidence: `internal/heddleengine/target.go` gives the codex-file row `ReRender: {"AGENTS.md","AGENTS.override.md"}`.
   - With D12's `CLAUDE.md` fallback, an edit to `CLAUDE.md` no longer re-renders `AGENTS.override.md`, so Codex reads stale repository instructions.
   - `CLAUDE.md` belongs in that row's `ReRender` list.

7. `LOW` — D11: the engine-block resolver must also serve `Table.CheckMarkers`.
   - Evidence: `internal/loomcli/darnwiring.go:266` calls `table.CheckMarkers(stencilsDir)` before any seat starts, and it reads every included block.
   - D11 names only seat composition and burler instruction composition as callers. If `CheckMarkers` reads an `engine-<block>` include without the seat's engine, every assisted table is refused.

**Paths traced, per decision**

- **D1**
  - Notice sender (no up line, no done line): survives.
  - Hold deferral: survives.
  - `QuietHold` (`seatengine/prompt.go:205`, `!isChair && !seat.Required`): survives, so the assistant holds quietly.
  - Stop of every seat at the chair's end: survives.
  - Probe/Resume with the marker: breaks (finding 1).
- **D2**
  - `ReviewResult` acceptance reads the chair's file only: survives.
  - The review-gate entry on the advisor seat: survives.
  - Archive-on-exit: breaks (finding 4).
  - Per-round recorded fans versus the assistant choice: breaks (finding 3).
- **D3**: per-round pick via `models.Pick`: survives. Mid-round edit: breaks (finding 3).
- **D4**
  - `SessionNamers` (no Codex namer; the empty tag stays on claude): survives.
  - `RecordReaders` (nil capabilities): survives.
  - Engine name equality test: survives once modelspec changes.
- **D5**
  - Lock plus atomic write across concurrent Prepares: survives.
  - Codex's own trust write: breaks (finding 5).
  - Prune of vanished roots: survives.
- **D6**: reed resume/revive fallback to `Cmd`: breaks (finding 2).
- **D7**
  - Creel `deliverOnce` for a strand without `CreelHooks` types reminders into the pane only when mail exists. Nothing mails the assistant, so this survives.
  - `post --interrupt` refusal: survives.
- **D8–D10**: no existing path rewrites their state: survive.
- **D11**: `CheckMarkers` in the darn and loom wiring: at risk (finding 7).
- **D12**: heddle re-render trigger: breaks (finding 6).
- **D13**: no other link writer: survives.

**Tool calls**

1. Grepped for `ResumeCmd`, `IsReportName` and `QuietHold`: to find reed's resume use and the seat quiet-hold rule, which my context lacked.
2. Read the reed resume loop, the lens-report readers and include readers: to confirm the empty-`ResumeCmd` fallback, which my context lacked.
3. Grepped for `CheckMarkers` callers and the review row's archive: my context lacked the pre-start marker check and the archive-on-exit set.
4. Read the burler review row's probe, fan-record and archive code, and `archiveStaleOutputs`: to see which files are archived and how per-round fans are recorded, which my context lacked.
