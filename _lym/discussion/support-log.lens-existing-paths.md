```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code paths that already write, reset or read each state the record introduces or changes, per decision.

**Paths traced**

D1 and D2, the store and its write path:
- The board's detached sync commits the whole Bolt checkout under the board lock (`Bolt.Commit`, then `commitWeftAt`, a wildcard stage). A written `heddle.json` is picked up. **Survives.**
- `Bolt.CommitWritten` and `Bolt.PullThenCommitWritten` commit only the paths they are given, under the Bolt write lock. A heddle write through them is serialized with pull and replay. **Survives.**
- Bolt's pull-first replay of a local `heddle.json` commit goes through D7's heddle replayer. **Survives.**

D3, the rendered files:
- `Fabric.Pull`'s warp reset leaves ignored files in place. **Survives.**
- The weft move's `git clean -f -d` is weft-only and has no `-x`, so ignored files are kept. **Survives.**
- Remove's dirty gate does not see the files, because they are excluded. **Survives.**
- `MerriamBase` reads the instruction files at `geom.WorktreeRoot`, which matches root placement, and now counts `CLAUDE.local.md`. **Survives**, but see finding 3.

D4, the appended prompt:
- `buildResumeCmd` reuses the fresh launch's file path. **Survives.**
- An adopted resume (`spec.ResumeSessionID`, set in `internal/loomshed/samplersession.go`) runs `Prepare` again. **Not settled by the record**, see finding 1.

D5, the exclude lines:
- `unseedGitExclude` (unwire, anchor change, `reconcile.go:906`) strips only junction-pattern lines and their bare names. `/CLAUDE.local.md` never matches. **Survives.**
- `seedGitExclude`'s legacy bare-name drop touches only junction names. **Survives.**
- An anchor change moves anchored junction lines only, so the root lines stay. **Survives.**
- `Healthy` and the drift checks read `WiredNames` only. **Survives**, provided the exclude list is never folded into the wired set.

D6, the warp seam:
- `PullHub` calls `Fabric.Pull` per beam, so it fires. **Survives.**
- A detached sync runs the same pre-run, so it fires. **Survives.**
- `ffWorktree` skips live pairs, so a live pair gets no re-render until its own pull. **Survives**, consistent with the record.
- Finalize's local parent move is not an origin take-in. The record states this. **Survives.**
- `CloneHub` makes `_board` (step 4) after the warp clone (step 2), so the "made" event must fire after step 4. See finding 4.

D7, the replay seam:
- Fabric-internal readers (`readEndedSet`, `localBoardEntries`, `pushclassified.go`, `pull.go`, the owner read in `originreconcile.go`) are covered by the run ledger. **Survives.**
- Readers outside fabric: see finding 2.

D9, the constant:
- `ConfigDirs`, the `.Board` readers, `StencilsDir`, `HubScratchDir` and `HubOwner` keep the same values today. **Survives.**

**Findings**

1. MEDIUM — D4: "a resume keeps the passages its fresh launch wrote" has no source on an adopted resume.
   - Evidence: `claudeengine.Prepare` runs again when `spec.ResumeSessionID` is set (`claudeengine.go:146`). `loomshed/samplersession.go:209` sets it in a fresh `Prepare` call, which may run in another run directory and another process.
   - The record doesn't say where that `Prepare` gets the earlier passages, or what happens if they are missing: re-read, or none.
   - Only reusing `Launch.ResumeCmd` keeps the file automatically.

2. MEDIUM — D7: packages outside fabric that use the seam D7 removes are not addressed.
   - Evidence: the writer's grep for `InstalledBoardReplay\|BoardEntry\b` matched production files `internal/battencli/owner.go`, `internal/battencli/wire.go`, `internal/driftengine/deps.go`, `internal/driftengine/seats.go`, `internal/loomcli/driftwiring.go` and `internal/shedrecipe/entries_drift.go`.
   - D7 says no board-named replay type is left in fabric. These readers need a stated replacement: the run ledger, or reading `boardengine` directly.
   - I did not read those files. Some matches may be same-named local types.

3. LOW — D3/D4: Webster's Merriam start base counts `CLAUDE.local.md` but not the `claude`/`prompt` passages.
   - Evidence: `MerriamBase` reads only the `InstructionFiles` names at `WorktreeRoot`.
   - A large prompt passage is missing from Merriam's partition and rebaseline pricing. The record is silent on whether that is acceptable.

4. LOW — D6: the record doesn't fix when "made" fires inside `CloneHub`.
   - Evidence: the `CloneHub` doc orders the warp clone (step 2) before the `_board` worktree (step 4).
   - Fired at warp-clone time, heddle would read no `heddle.json` and render nothing. The decision should say it fires after the Bolt checkout exists.

**Tool calls**
- Grepped for `git clean` calls and read webster's instruction-file use (`auditengine.go`, `runlevel.go`, `rebaseline.go`). My context had no reader of `InstructionFiles` beyond the listing.
- Read `MerriamBase`'s file read, the weft-move clean in `destroy.go`, and the resume lines in `claudeengine.go`. I needed where the instruction files are read, whether the clean touches ignored files, and how resume is set up.
- Read the resume branch in `claudeengine.go`, the setters of `ResumeSessionID`, and `Bolt.Commit`'s staging. These test D4's keep rule and D2's commit path.
- Read `CloneHub`'s doc for its step order, which my context lacked for D6's "made" timing.
