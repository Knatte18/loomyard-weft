# Support log — test-tmp-cleanup

## Interview

Autonomous run: no operator answered, so each turn is the writer's own exploration and pick.

1. Board read. Task merges the temp-leak bug with `lyx-cleanup`. Related entries read: `sed-read-ok` (Webster fork `/tmp` use, a stencil concern, not this task), `doctor`, `proc-child-lifetime`, `config-layers-run-settings` (adds `.lyx` local override files; D9–D11 never touch `.lyx/config`).
2. Leak sources checked against code. `loomcli-smoke-lyx-*` was removed in `b7d4fd0ba`. Nearly every `TestMain` already calls `tmuxkit.Main`, which redirects `TMPDIR` to a private directory and removes it, so in-run fixtures already go; the live leaks are `HermeticGitEnv`'s gitconfig (created before the redirect; 1263 in `/tmp` today) and directories of killed binaries. Picked a locked per-process run directory under one root (D1–D4).
3. Liveness: pure age would delete a long gate run's directory; picked an OS file lock plus a creation grace (D2).
4. Scan shape: banning bare system-temp creation outright instead of matching each site with a cleanup (D6).
5. `lyx cleanup` segments: found `lyx board prune`, `lyx fabric prune`, `lyx fabric cleanup`, logger's exit sweep. Fabric's verbs hold no lock against `fabric add` (no lock in `prune.go`/`cleanup.go`), so they stay out (D8).
6. `.lyx` disposability: prime holds `.lyx/shed/<slug>` mirrors (108), `.lyx/batten/*.log`, `.lyx/orch/handoffs` (44), logs (already retention-swept), one shuttle dir. Picked batten mirror/log, orch handoffs beyond newest, logs via existing sweep; everything else left (D9–D11).
7. `.scratch/` has no owning module; the cleanup module owns that segment with a 14-day age (D11).
8. Lens review: all six forks (premises, goal-scope, board-fit, pattern-fit, scenarios, leanness) returned. Findings, duplicates merged:
   - board-fit + scenarios BLOCKING, nested child sweep deletes the parent's lock-less fixtures: fixed; D3 puts a child beside its parent, D2 sweeps only prefixed run directories.
   - goal-scope MEDIUM, D8 premise false (add creates warp first): fixed; D8 re-derived against `pairteardown` resuming half-removed pairs, which archives the weft tip.
   - goal-scope MEDIUM, Goal claims "every module's cleanup": fixed; Goal narrowed to D7's segments.
   - goal-scope + pattern-fit + premises MEDIUM/LOW, `os.TempDir()` ban hits non-creating uses: fixed; D6 narrowed to creation calls, Scope Out names them.
   - pattern-fit MEDIUM, `PATTERN-test-isolation` background and `tmuxkit` doc not named for update: fixed in Scope.
   - pattern-fit MEDIUM, temp package's own leaf section unstated: fixed in D1 and D4.
   - premises MEDIUM, `.lyx/batten/*.log` has no owner: fixed; dropped from D9, listed in Scope Out.
   - premises MEDIUM, log segment used defaults over `logger.yaml`: fixed in D7.
   - premises MEDIUM, D3 contradicted tmuxkit's beside-not-inside nesting: fixed by D3's rewrite.
   - premises MEDIUM + scenarios LOW, orch doc says all handoffs kept, phase race, "newest" undefined: fixed in D10.
   - scenarios MEDIUM, run lock lives inside the mirror D9 deleted: fixed; D9 keeps the lock file and empties the rest.
   - scenarios MEDIUM, interrupted apply not resumable: fixed; tracked-but-deleted run directories are candidates.
   - scenarios MEDIUM, Windows `tmuxkit.Main` returns early: fixed in D4.
   - scenarios MEDIUM, root trust boundary: fixed in D1 with an acceptance scenario.
   - scenarios LOW, malformed status, grace clock, `.scratch` links, gate children after SIGKILL: fixed in D9, D2, D11, D5.
   - board-fit MEDIUM, `.scratch/` holds evidence running tasks cite: fixed; D11 skips paths cited by not-done board entries.
   - board-fit LOW, overlap with `reed-socket-test-fd-sweep`: fixed in Open risks.
   - board-fit LOW, `orch-split` relation: fixed; D7 bound says no row calls it.
   - board-fit LOW, `tokencount-cutoff-skew` relevance: rejected; slug reuse is already possible by hand, and the note tracks that risk itself.
   - goal-scope LOW, panic departure unstated: fixed in D4.
   - goal-scope LOW, legacy leftovers left to the operator: rejected; reason in Rejected alternatives.
   - goal-scope LOW, "orch notes" reading unstated: fixed in Auto-mode assumptions.
   - pattern-fit LOW, `kitBase` reads `TMPDIR` against D1's sole-resolver claim: fixed; claim dropped, detection moves into the temp package (D3).
   - pattern-fit LOW, D6 PATTERN section undecided: fixed ("Paths and geometry").
   - pattern-fit LOW, fabric destruction chokepoint and refusal-spec reading: fixed in D9 and D7.
   - premises LOW, `cmd/lyx` test-budget `TestMain` is a string fixture: fixed; the stale list is cut.
   - premises LOW, scan must parse the AST, not text: fixed in D6 and acceptance.
   - leanness LOW/MEDIUM, restated PATTERN constraints, duplicate docs bullet, filler assumptions, D1/D6 overlap, D3 `go build`, acceptance overpromising on reuse: fixed by cutting or rewording each.
   - leanness LOW, smoke-lyx fact misfiled: fixed; moved to Notes.

## Rejected alternatives

- Age-only sweep of temp leftovers: deletes a live long run's directory; lock liveness is exact.
- Sweeping legacy `/tmp` prefixes (`gitkit-gitconfig-*`, `hubforge-bare-*`, …) from `lyx cleanup`: after landing no code creates them, the remainder is small, and a hard-coded prefix list (including tmuxkit's bare `lyx<digits>`) risks matching foreign files.
- Pairing each bare `MkdirTemp("")` with a detected cleanup in the scan: hard to check statically and still leaks on a kill.
- Calling `lyx fabric prune`/`cleanup` from `lyx cleanup`, or only while no batten run is live: they can pre-empt a half-finished batten teardown, and with batten running continuously the guarded form would almost never run.
- Banning every `os.TempDir()` call in the scan: it hits non-creating uses and goes past the board's ask.
- Deleting a done slug's whole `.lyx` scratch directory: its run lock lives there, so deleting it under a held lock lets a concurrent run lock a new inode.
- A nested child creating its run directory inside the parent's: lengthens socket paths and puts the parent's live files in the child's sweep.
- Cleaning shuttle run directories: not asked for, small in the prime, and pairs' go with the pair.
- `lyx cleanup` shelling out to `lyx <module> cleanup`: in-process calls give one envelope and no spawn.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Temp root liveness — auto-pick: OS file lock plus 10-minute creation grace (D1, D2).
- Q2 Where once-per-binary fixtures are removed — auto-pick: with the process run directory, by `tmuxkit.Main` or an explicit `TestMain` removal (D4).
- Q3 Scan rule — auto-pick: ban bare system-temp creation outside the temp package, empty allowlist (D6).
- Q4 Does `lyx cleanup` run fabric prune/cleanup — auto-pick: no (D8).
- Q5 Which `.lyx` paths are disposable — auto-pick: batten's run scratch contents (lock file kept) of a cleaned or aged orphan slug, orch handoffs beyond the newest, logs through the existing retention sweep (D9–D11).
- Q6 `.scratch/` age — auto-pick: 14 days, matching logger's default retention.
- Q7 Where `lyx cleanup` runs — auto-pick: prime only, as batten does.
- Q8 Legacy `/tmp` leftovers — auto-pick: not swept; removed once by hand.
- Q9 `.scratch/` evidence cited by board entries — auto-pick: skip a path a not-done entry cites (D11).
- Q10 Orch cleanup during a handoff cycle — auto-pick: idle phase only (D10).
