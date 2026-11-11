```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens over the claude-account-weights decision record.

**Walkthrough list**

- **D1.** Case: the machine file lists A and B, and the worktree's tracked `claude.yaml` also lists A. The refusal can't see the tracked value. **Purpose not met.**
- **D2.** Case 1: worktree on account B (non-default), the operator removes the record, and the next launch is the orch's resume. The resume check refuses before any search runs. **Not met.** Case 2: a fresh launch with no record. `Prepare` gets no session ids to search for. **Gap.**
- **D3.** Case: weights 2 and 1, counts starting at 0. Six picks give A B A A B A: four for A, two for B, never two B in a row. **Met**, except the seed of the very first pick is undefined (LOW).
- **D4.** Case: a fresh task launch with the record on B. The engine's command ends the pane line, after the `env` module's exports. A reed relaunch replays the stored line. `OpenLaunch` chains an export the same way. **Met.**
- **D5.** Case: a session on A whose status line input has both windows. The settings carry `lym <verb> A`, and the verb merges into `usage/A.json`. **Met**, with an open cwd question (LOW).
- **D6.** Case: `lym report` on a landed run that ran on B. The reader is built with no config dirs and is handed only the worktree path. It can't learn the list, the `env` module or the hub's records. **Not met.**
- **D7.** Case: the prime orch's first launch after the change is a resume, with its transcript under `~/.claude`. The resume check falls back to the default and passes, and the record gets the default. **Met.** With a removed record on a non-default directory, the D2 case 1 failure applies.
- **D8.** Case: the orch watcher waits out a limit, then resumes through the stored resume line, which carries the export. **Met.**
- **D9.** Case: the operator links `<dir>/plugins` to `~/.claude/plugins`. Deploy's writes under `~/.claude/plugins/cache` reach both accounts. **Met.**

**Findings**

1. **MEDIUM · design · D2, "A resume launch with no record" (also D6, D7)**
   On a resume with no record, the resume check runs before `Prepare` and only reads the fallback directory. So a transcript in a listed non-default directory is refused before D2's search and record write can run.
   - Evidence: `internal/shuttleengine/run.go:581` calls `checkResume` before `launch`, and `engine.Prepare` runs at `run.go:642`. `CheckResume` resolves its directory with no record to go by (`resume.go:29-38`; D6: "A reader with no record takes the `env` module's value, then the process's, then the default").
   - Concrete break: an orch on account B whose record the operator removed, which is D2's own way forward and the Auto-mode assumption. Its next resume is refused with "no transcript", and nothing gets recorded.
   - The record should state that the resume check searches every known directory when there is no record, or that the record is resolved before the check runs.

2. **MEDIUM · design · D6, the `lym report` / `tools/tokencount` bullet and the "known directories" list**
   The readers can't reach the known directories with what they are handed.
   - Evidence: `internal/shuttleengines/shuttleengines.go:222-228` builds `RecordReaders()` with `configengine.Dirs{}`. `LocateSessions(worktree string)` gets only a path (`shuttleengine/transcriptread.go:138`).
   - With no `Board`/`Anchor`, the machine layer never stacks (`configengine/doc.go:54-60`). So the engine can't load `claude.accounts` or the `env` module.
   - Enumerating "the hub's worktree records" needs hub geometry, and an engine is told its geometry and never derives it (`PATTERN-cwd-resolution`, Told geometry).
   - Other callers of `RecordReaders()` hit the same gap: tokencount (`tools/tokencount/reads.go:42`) and the run usage recorder (`internal/loomcli/wiring.go:640`, `internal/shedadapters/sessionusage.go:107`).
   - Result: D6's acceptance criteria for a landed run on a non-default account can't be met by the mechanism as stated. The record has to say how the readers get the hub's dirs, for example `RecordReaders` taking the hub's config `Dirs`, and how the worktree records are enumerated without the engine deriving geometry.

3. **MEDIUM · design · D2, the fresh-launch search ("the session ids its shuttle run records and reed strands hold") and D3 step 1**
   The search runs inside the claude engine's prepare (D4), but `Prepare(runDir, spec, cfg)` gets no list of the worktree's session ids.
   - Evidence: `claudeengine.go:121`. `Spec` carries one `SessionID` (`shuttleengine/engine.go:50`). Reed's strand ids live in `internal/reedengine/state.go:29`.
   - The record doesn't say where the engine gets the ids, or whether claudeengine may read reed's and shuttle's records. That input is what the search's correctness rests on.

4. **LOW · design · D1, "refused when the module loads, told by the layer provenance `configengine` reports for each unit"**
   The provenance it names reports only the highest layer.
   - Evidence: `configengine/layers.go:589-593` (`Origins`: "the highest layer holding each top-level key") and `layers.go:626-686` (`Provenance` labels the highest layer).
   - With the machine file also listing accounts, a tracked `claude.yaml` holding `accounts` is masked and never refused. The tracked-only acceptance test would still pass.
   - The mechanism should be stated as a per-layer read of the stack's `Layers`.

5. **LOW · consistency · D3, the seeding sentence**
   "Starts at its weight times the lowest count over weight among the listed accounts that have a count" is undefined when no listed account has a count, which is the hub's very first pick. The record should state that the start is 0 there.

6. **LOW · design · D5, the status line verb**
   The verb has to resolve the hub (the usage file under `<Bolt>/.lym`, and the list to refuse unlisted names). The record doesn't say which directory Claude Code runs the status line command from.
   - Evidence: the creel hook commands `cd "${CLAUDE_PROJECT_DIR:-.}"` before running `lym` (`claudeengine/settings.go:117`), which suggests the cwd can't be trusted.
   - `lymcwd.Resolve` needs the cwd to equal the anchor (`PATTERN-cwd-resolution`).

**Tool calls**

1. Grep of `Spec` session fields, `envconfig` importers and the hook command form: to see what `Prepare` is handed and how a lym hook names its binary.
2. Grep of reed strand session-id fields and `Prepare`'s caller: to find where the D2 search's input lives and the order of the launch steps.
3. Read of `run.go` around `checkResume` and `launch`: to confirm the resume check runs before `Prepare`.
