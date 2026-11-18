```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker review of the decision record's claims, bounds and ordering promises.

**Findings**

1. **BLOCKING — D6 (and D9, D7):** D6 assumes the resume check runs after the engine copies the transcript, but it runs before.
   - Evidence: `internal/shuttleengine/run.go` calls `r.checkResume` at line 589, before `engine.Prepare` at line 650. D6 puts the copy in "when the engine prepares a resume" and says "the resume check then runs as today".
   - Counterexample: a worktree switches from B to A, and its session's transcript exists only under B. `CheckResume` stats the resolved directory A and returns `ErrNoTranscript` before `Prepare` can copy.
   - On that error, `internal/orchcli/start.go` `launchOrch` "launches fresh" and names the session as abandoned. So D9's "orch sessions started before the change resume … through D6's copy" fails, and the orch silently loses its conversation. D7's switch-and-resume of shuttle runs fails the same way.
   - Fix: the copy has to happen inside `CheckResume`, or before it.

2. **BLOCKING — D6:** a stale copy already in the target directory blocks the copy, and the resume loses turns.
   - D6 copies only when the transcript "is not under the resolved directory".
   - Counterexample: the worktree is on B, hits a limit and switches to A (copy B→A), and the session goes on under A. Later A runs out with B's reset passed, so it switches back to B (or the operator runs `account set B`).
   - B still holds the old transcript, so nothing is copied. The resume starts from B's stale file and drops every turn made under A.
   - D8's "most recent copy is a superset" also breaks once the copies diverge.
   - Fix: copy whenever another known directory holds a more recently written copy.

3. **MEDIUM — D7 and D5:** a worktree with no entry, running on the default directory, can loop.
   - Setup: a worktree made before the change, with no entry, runs on `~/.claude`, and `~/.claude` is listed as account X.
   - The session hits a limit. Its "launched under" account is none, so D5 has no account name and records no `blocked_until` for X.
   - D7's switch excludes only "the one the session was launched under", which is none, so it can pick X. X differs from none, so the strand is stopped and resumed on the same exhausted directory. The next limit hit repeats this.
   - Fix: map the launch directory to a listed account by cleaned path, both when D5 records a hit and when D7 excludes the current account.

4. **MEDIUM — D7:** the record says "stops the strand through its stop verb and starts it again as a resume", but the stop verb ends the run.
   - Shuttle's stop verb records the run as stopped (`runOutcomeStopped`, behind the completion-signal rule). The producer waiting on that `*Run` then gets a negative outcome instead of a run that continues.
   - The record doesn't say the restart stays inside the same run's Wait, as a respawn the way the API-error retry works, or how the caller's run handle follows the new strand.
   - As written, a mid-phase switch can end a phase as stopped.

5. **MEDIUM — D2 and D3:** a recreated worktree inherits a stale entry.
   - The entry is keyed by worktree name, and "a worktree that already has an entry keeps it". Pruning runs only at a pick, and only for directories already gone.
   - Counterexample: a task slug's worktree is torn down and recreated before any other pick runs. The recreated worktree keeps the old entry and is never weighed again. That skews the share, and it can sit on an account marked blocked.
   - The record states no bound for this.

6. **LOW — D9 and D7:** D7 contradicts D9 for the prime.
   - D9 says the prime "has no entry until the operator sets one". D7's orch-watcher switch writes an entry for an unpinned prime when the orch hits a limit.
   - Unstated either way.

7. **LOW — D5:** the merge rule doesn't say it keeps `blocked_until`.
   - The status line verb's merge is described window by window. If the rewrite drops `blocked_until`, the next status line tick after a limit hit (lower or absent figures) makes the account eligible again.

8. **LOW — D7 bound:** any agent in a worktree pane can run `lym claude account set`.
   - The stated bound covers only the scope of the change (a local entry, no pin override). It doesn't say that agents, not only the operator, can move their own worktree.

**Claims attacked**

- **Held:**
  - D4 ordering: the export comes after the `env` exports, and the unset wins over an `env` value.
  - D4 scrub: panes take their environment from the scrubbed server, not from the calling lym process.
  - D3 "each worktree adopt adds": `internal/adoptcli/foreground.go` calls `AddPair` for both beams (line 124) and taken-over runs (line 326).
  - D3 concurrent picks, since the entry is re-read under the lock.
  - D7 concurrent limit hits making one switch.
  - D1 pin bound.
  - D5 forged-input bound.
  - D2 behaviour when the file is wiped.
- **Broken:** D6 copy-then-check ordering (finding 1), D6 when the target holds a stale copy (2), D7 exclusion for a worktree with no entry (3), D7 restart through the stop verb (4), D2 stale entry (5), D9 "prime has no entry" (6), D5 `blocked_until` surviving the merge (7).

**Tool calls**

1. Grepped `internal/orchcli/start.go` and `transcriptread.go`, to learn how the orch resume reaches the engine; my context had only the subagent summary.
2. Read `internal/orchcli/start.go` around lines 730–800 and grepped `apierrorretry.go`, for `launchOrch`'s fallback on `ErrNoTranscript`.
3. Read `internal/shuttleengine/run.go` around lines 955–1000 and grepped it, to confirm `checkResume` runs before `Prepare`.
4. Grepped `internal/` for worktree-creating code, to test D3's list of pick points.
5. Grepped `adoptcli` and `battencli`, to confirm adopt's `AddPair` covers taken-over runs.
