```yaml
lens: bound-breaker
outcome: reported
```

## Findings

**1. BLOCKING — D2/D3/D7: worktrees already in flight when the change lands get a fresh pick on their next resume, and the resume then fails.**
Counterexample: the prime's `ly:orch`, a beam orch or a running task's driver was started under `~/.claude` by the old binary; there is no `.lym/claude/account.json`. After the upgrade its next launch is a `--resume`. D2 counts this as the "first session", so D3 picks by counter — with weights 2:1 the second account comes up ~1/3 of the time. `buildResumeCmd` then exports that account's dir, and `CheckResume` refuses: "no transcript for session … only a session run from this directory can be resumed" (`internal/shuttleengine/claudeengine/resume.go`, `checkResumable`) — no fallback to a fresh start. Reed's stored `ResumeCmd` (`internal/reedengine/state.go`) for a strand from the old binary has no export, so a reed relaunch of that strand stays on the default dir and the worktree's sessions split across two accounts. Fix: a resume launch with no record makes no pick and records the directory that holds the session's transcript (the default, for pre-upgrade sessions); a pick happens only on a fresh launch.

**2. MEDIUM — D2: re-pick after an account is removed from the list makes a resume refused, not just "logged".** The record says the re-pick is "logged" but not that a resume under the new pick hard-fails in `CheckResume`. Same counterexample as #1: removing an account while its worktree's orch/dobby/driver is resumable breaks the resume. State the bound, or make the re-pick apply only to fresh launches and make a resume naming a removed account fail with a way forward.

**3. MEDIUM — D1/D3/D5: "machine" accounts, counter and usage snapshots are per hub, not per machine.** This machine has four hubs (loomyard, lym-csharp, lym-test, scribe), each with its own `_bolt`. `claude.machine.yaml`, `account-picks.json` and `usage/<account>.json` all live under `<Bolt>/.lym`, so each hub has its own weights and its own headroom view of the same account. The weighted share holds only within a hub, and one hub's sessions pushing account B to its limit never update the other hubs' snapshots — they read B as having headroom until their own session on B reports. The record calls accounts "facts of the machine" but stores them per hub, silently. Fix: put the usage snapshots (and the counter, if wanted) in a per-user machine-wide location, or state the per-hub bound and that the config is repeated per hub.

**4. MEDIUM — D5: concurrent status line writers regress figures.** Several sessions on one account each run the verb ~every refresh. Session X holds figures from its last API response (40%); session Y just saw 88%. X's next refresh writes 40% "since changed" vs the file, and the account reads as having headroom. "Only when changed" plus atomic rename gives no ordering. Fix: per window keep max `used_percentage` for the same `resets_at`, or the newer `resets_at`.

**5. LOW — D5: precedence reason is wrong.** "the worktree's project settings take precedence over user settings" — the engine passes settings by `--settings <runDir file>` (`buildLaunchCmd` takes `settingsPath`), not via project settings. Outcome holds (flag settings also outrank user settings), but the stated reason is wrong.

**6. LOW — D1: tracked-layer refusal needs layer provenance.** Refusing a value from a tracked layer requires knowing which layer a stacked value came from; `configengine` hands the module the merged result. The record states a refusal whose mechanism may not exist — unverified; the plan writer should confirm provenance is available or add it.

**7. LOW — D1/D2: standalone runs unaddressed.** Outside a hub, Prepare's Board dir is empty, so the machine layer never stacks and no accounts load. Not stated that standalone runs keep today's behaviour.

**8. LOW — D9/D3: credentials stat gate passes an empty file.** Automated tests plant an empty `.credentials.json`; in real use a stale/expired login also passes the stat. The gate proves existence, not a login — the record calls it "logged-in". Minor, but the bound should be stated.

## Claims attacked

| Claim | Result |
|---|---|
| D4 export after env module wins | Held (`spawn.go` chains `paneEnvExports` ahead of `launchCmd`) |
| D4 reed relaunch keeps account | Held for new strands; broken for strands stored by old binary (#1) |
| D2 resume/re-attach/hand pass keep account | Broken across the upgrade (#1) |
| D2 removed account re-picked | Broken on resume (#2) |
| D3 one pick per worktree under lock | Held |
| D3 long-run share | Broken across hubs (#3) |
| D5 reset time bounds staleness | Held |
| D5 recorded figures are latest | Broken under concurrent writers (#4) |
| D5 settings precedence | Outcome held, reason wrong (#5) |
| D1 tracked-layer refusal | Mechanism unverified (#6) |
| D6 registry/transcripts move with account | Held per docs fact |
| D8 limit wait keeps account | Held |
| No-accounts keeps today's behaviour | Held |
| Standalone | Unstated (#7) |

Tool calls:
- `internal/reedengine/panebin.go` 55–130 and grep of reed launch/resume cmd — to check export order and that reed replays a stored `ResumeCmd`; context had only the subagent summary.
- `ls` of hubs / `hubgeom.paneEnv` / engine env file — to check whether this machine has multiple hubs and what Prepare's env file holds.
- grep `resume.go` functions — to find the resume check's behaviour on a missing transcript.
- read `resume.go` 20–100 — to confirm a missing transcript refuses resume rather than starting fresh.
