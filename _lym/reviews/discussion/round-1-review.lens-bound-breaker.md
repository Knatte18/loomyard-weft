```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break every bound, guard, guarantee and claim about existing code in `decision-record.md`. Two claims break at BLOCKING, three at MEDIUM, three at LOW and one at NIT.

**Claims attacked**
- D1: unknown keys survive set, add, remove and replay through D1's types — held.
- D1: an absent file holds no entry — held.
- D2: a write saves under the `BoardWriteLockFile` lock, then the board's detached sync commits — held (`internal/boardengine/sync.go:72` commits through `bolt.Commit`).
- D2: a write never touches a live worktree — held.
- D3: Claude Code reads `CLAUDE.local.md` from the working directory and each parent — held (memory docs, "How CLAUDE.md files load").
- D3: rendering `CLAUDE.local.md` leaves the repository's own instructions reaching Claude — broken (finding 2).
- D3: "two renders of one worktree write the same text" — broken (finding 4).
- D3: the target-off-exclude-list refusal means "a render never shows as an untracked file in warp" — broken (finding 3).
- D3: the marker check "covers a file the repository tracks" — broken (finding 6).
- D3: the HTML-comment marker is harmless in context — held (the docs say block-level comments are stripped).
- D4: the append file is "echoed nowhere" — held (only `claudeengine.go:227-232` writes it, and nothing reads it back).
- D4: a resume through the launch's own line reuses the file — held.
- D4: a missing seam or no hub means no passage — held.
- D5: a listed line is never unseeded — held (`reconcile.go:906` unseeds only stale junction names).
- D5: the same mutation and lock as junction seeding — held (`gitexclude.go:52-67`).
- D6: a detached sync reaches the seam through the same pre-run — held (`cmd/lym/main.go:136-152`).
- D6: a seam failure is "reported in the verb's result" — broken for the detached path (finding 5).
- D6: "made" fires last — held.
- D7: the ended-run set-aside is narrowed to the ledger part — held as stated.
- D7: a non-ledger set-aside is "never under a bare entry slug that could read as a run's" — broken (finding 7).
- D7 acceptance: no `done` status left in fabric's replay or ended-set code — broken literally (finding 9).
- D9: `lymcwd`'s import set and `fabricengine` importing `runproc` — held.
- D9: `runproc.HubOwner` is set from the `lymcwd` constant — broken against PATTERN-leaf-packages (finding 1).
- D9 acceptance: the `_board` literal appears in one Go declaration — broken by the policing test (finding 8).

**Findings**

1. **BLOCKING · design · D9, second bullet.** `runproc.HubOwner` being "set from the `lymcwd` constant" requires `internal/runproc` to import `internal/lymcwd`.
   - PATTERN-leaf-packages limits `runproc` to the standard library, `internal/proc`, `internal/state` and `internal/logger`.
   - The record declares no change to that entry, and D9's own import argument only rules out `fabricengine` as the source.
   - Evidence: `pattern/PATTERN-leaf-packages.md:82-85`; `runproc`'s current imports are `logger`, `proc` and `state`.

2. **BLOCKING · design · D3 (`claude-file` → `CLAUDE.local.md`), Open risks line 3.** In a repository whose team instructions live in `AGENTS.md` with no `CLAUDE.md`, heddle's render makes every Claude session stop reading the team's `AGENTS.md`.
   - Under Claude Code's default `claude-md-or-agents-md` setting (v2.1.277 and later; v2.1.316 is installed here), a `CLAUDE.local.md` in the working directory or above "counts". Claude then reads "your `CLAUDE.md` files only".
   - The docs say: "adding one … in a project that relies on `AGENTS.md` stops Claude from reading `AGENTS.md`".
   - This breaks the board entry's rule that the team's instructions are not hidden. The record never states the fact or bounds it, even though it cites these same docs.
   - Open risks leaves the `CLAUDE.local.md` behaviour to be found out later. The docs answer it now.
   - Evidence: code.claude.com/docs/en/memory, sections "AGENTS.md" table, "When Claude Code reads AGENTS.md" NOTE, and "Choose which instruction files load".

3. **MEDIUM · design · D3, last render bullet, with Open risks line 4.** The refusal checks fabric's `warp_exclude` config, not the lines actually seeded in `.git/info/exclude`.
   - Counterexample: a worktree that predates the landing has no seeded lines. The config default lists both files. Open risks' own remedy, `lym heddle render`, then writes `CLAUDE.local.md` as an untracked file in warp.
   - D5 seeds only on create, re-wire or `lym fabric reconcile`, and D3 requires none of them first.
   - Evidence: decision-record D3, D5 and Open risks; `junction.go:703` (seeding happens only on the wiring paths).

4. **MEDIUM · design · D3, first render bullet.** "Two renders of one worktree write the same text" fails when `heddle.json` or `AGENTS.md` changes between the two renders' reads.
   - D6 says a re-render takes `heddle.json` "as it stands then", and a Bolt replay or pull changes it.
   - Counterexample: a detached sync's took-in render overlaps a hand `lym heddle render`. Both truncate and write in place, so the shorter text lands over the longer text's tail and the file is corrupt.
   - A session starting mid-write also reads a partial file.
   - The no-temp-file rule removes the atomic write, and no lock bounds this.
   - Evidence: decision-record D3 and D6; support-log rejected alternative "atomic write".

5. **MEDIUM · design · D6, seam-failure rule.** A downgraded guard's stated bound, "reported in the verb's result with `lym heddle render` as the way forward", reaches no one when the verb is the detached sync the same Decision names.
   - A failed took-in render, for example on a `heddle.json` that fails to decode after a replay, leaves a stale `AGENTS.override.md` hiding the team's new `AGENTS.md`. Only a log line records it.
   - Evidence: decision-record D6 ("A detached sync is a lym process…"); `boardengine/sync.go:10` (the sync runs detached).

6. **LOW · design · D3, second render bullet.** The marker's presence does not prove the file is untracked.
   - A heddle render committed by accident, for example after finding 3, is tracked and carries the marker.
   - Heddle then rewrites it on every render and removes it when its target has no entry, so a tracked file shows as modified or deleted.
   - Evidence: decision-record D3, "which covers a file the repository tracks".

7. **LOW · consistency · D7, set-aside bullet.** Tags have the shape `set-aside/<slug>/<short sha>`.
   - A replayer named `heddle` produces `set-aside/heddle/…`, which matches a run slug `heddle`.
   - Nothing reserves replayer names against slugs, so the "never reads as a run's" bound is unbounded.
   - Evidence: `internal/fabricengine/replay.go:483-491` (`setAsideTag`).

8. **LOW · consistency · D9 acceptance.** "The `_board` literal appears in one Go declaration" cannot hold: the geometry-literal test must name the token it polices.
   - Evidence: `internal/lymcwd/enforcement_test.go:209` and `:225`.

9. **NIT · consistency · D7 acceptance.** "No … `done` status … left in fabric's replay, ended-set … code" is false as written: the ended-set narrowing reads the run's own status `"done"`, which is not a board status.
   - The criterion should say "board status".
   - Evidence: `internal/fabricengine/replay.go:289` (`finalizeEntered`, used by `withoutFinalizeEntered` at :321).

**Tool calls**
1. `reconcile.go:880-915` and `replay.go:278-330`: the stale-exclude unseed path and the `"done"` site in the ended-set narrowing; the manifest had only grep hits for them.
2. Readers of `appendPromptText`, the root pre-run's seam installs, and `runproc`'s imports and leaf rule: needed for the D4 "echoed nowhere" claim, the D6 detached-sync claim and D9.
3. `commitDirty`'s commit call and `setAsideTag`'s format: needed for the D2 sync claim and the D7 tag bound.
4. `PATTERN-leaf-packages.md:82-92`: `runproc`'s closed import set, which the manifest did not include.
