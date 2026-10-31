```yaml
lens: bound-breaker
outcome: reported
```

Fork lens: bound-breaker. I attacked every claim and bound in the record.

**Claims attacked**
- Constraints fact on the watchdog and Open risks bullet 2 (D7 for `Beam-Await` and `PR-Review`): broken. The code answers the question now, and the stated mechanism is wrong (F1).
- D6: "a return to the row after it ended Done, by Finalize's reroute … raises it": broken. A reroute never reaches the row (F2).
- D6: the entry stamp keeps a stale flag from ending a re-entered row: broken by a re-entry that has no Done between visits (F3).
- D6 bound: "a dirty checkout never ends the row, so Describe and Finalize always start from a committed tree": broken. The order of the check and the stop is unstated, and each order leaves a hole (F4).
- D4 bound: "Describe shows every added entry": broken (F5).
- D7 bound: "as batten's Run-Shed step already is": broken as cited (F6).
- D1: PR-Review passes with no pull request on an empty landing: held (commit `5f444b56`).
- D2: a profile with no block for the recipe is refused at seed: held (`internal/battencli/runsettings.go:17`, `internal/runprofile/runprofile.go:82`).
- D3: a Finalize reroute with the perch off halts for the orch: held, if `rerouteReviewed` reads the gate (`internal/landingshed/finalize.go:753-756`, `792-796`). `CodeReviewOff` is read only by Parent-Align (`internal/landingshed/parentalign.go:435`), a row sampler lacks, so naming it is harmless.
- D5: an unawaited start has no deadline: held (`internal/shuttleengine/run.go` `StartSession` doc).
- D6 bound: the verb can be run from the operator's shell: held. The verb it is modelled on takes no arguments and acts on the cwd worktree, with no caller rule (`internal/loomcli/prcontinue.go:66`).
- D6: a file event cuts a sleep short: held. `internal/fswatch` exists over fsnotify (`go.mod:7`). An implementation must watch the directory, because a `state` write replaces the file.
- D9: one tmux server per hub path, and bindings re-pinned from a newer build: held (`internal/reedengine/server.go:55`, `internal/reedengine/bindings.go:4`).
- Constraints fact on the `decision add` caller rule: held (`internal/loomcli/decision.go:8-9`).

**Findings**

F1. MEDIUM · design. Location: Open risks bullet 2, and the Constraints fact on the watchdog.
Summary: the record leaves for later a fact the code answers now. Today neither `Beam-Await` nor `PR-Review` is exempt from the watchdog. The Constraints fact asserts that exemption while the Open risk says it is unknown, and the fact cites the wrong mechanism.
Evidence:
- `agentActivityFor` (`internal/loomcli/looparm.go:132-187`) has no row exemption.
- Neither wait writes its trace on a wake. `beamawait.go` and `prreview.go` hold no `logger.Debug`, and their wake loops log only failures (`internal/landingshed/prreview.go:385-470`), so the trace mtime goes stale.
- `PR-Review`'s session is started through `StartSession`, which never calls `Wait` (`internal/shuttleengine/run.go:515-524`). No wait marker exists for it, held or not: markers are written only from a `Wait` path (`internal/shuttleengine/waitmark.go:239`). So "an interactive session waiting on the operator is held" is the wrong reason.
- The session counts as activity only through transcript and event mtimes (`internal/shuttleengine/activity.go:149-178`).
- So a `Beam-Await` step is killed after `step_idle_timeout_min` of member silence, and a `PR-Review` step after that long with no reviewer or agent activity.

Fix: record this answer and its evidence under Constraints, and drop the Open risks bullet.

F2. LOW · consistency. Location: D6, entry-stamp bullet, and the matching D6 acceptance criterion.
Summary: "by Finalize's reroute" cannot happen. Finalize's `on_reroute` is `Code-Bouncer` (D1; `contracts/recipes/darn-recipe.yaml`, Finalize row), so a reroute never re-enters `Sampler-Session`, and the acceptance case "a done flag written before a reroute … back to the row" has no path to test.
Fix: drop the reroute from both places.

F3. MEDIUM · design. Location: D6, the entry stamp (the count of the row's Done entries).
Summary: a re-entry with no Done between the two visits keeps the stamp, so a stale flag ends the new visit.
Counterexample:
1. The run halts at `Sampler-Session`.
2. The operator runs `lym loom sampler-done`, which the D6 bound accepts while the run is halted.
3. Instead of resuming, they run `lym loom goto Record-Write` or `goto Preflight`.
4. Re-entry appends no Done entry for the row (`internal/shedengine/run.go` `appendOrFold`; Done entries never fold).
5. The flag's stamp still equals the current one, so the first wake ends the new visit Done on a clean checkout, before the operator works with the fresh session.

This defeats the stamp's stated purpose (support log, Turn 5).
Fix: count entries into the row, or clear the flag on each entry, and add an acceptance criterion for a goto with no Done.

F4. MEDIUM · design. Location: D6, the wake that reads a done flag.
Summary: the record does not say whether the checkout is checked before or after the session is stopped, and each order breaks the guarantee.
- Check, then stop: the agent's turn goes on after the verb returns, and a write between the check and the stop reaches Describe as a dirty tree.
- Stop, then check: a dirty result must resume the session. D6's dirty branch only "mails the session … and keeps waiting", which mails a stopped session.

`PR-Review` stops its session before the deciding read and resumes it when the review goes on (`internal/landingshed/prreview.go:446-466`).
Fix: state stop-then-check, with a resume on a dirty result before the mail, and add an acceptance criterion.

F5. MEDIUM · design. Location: D4, Bound.
Summary: one leg of the bound on the unverified `--by operator` label is false. The Describe stencil reads the record's `## Goal` as scope amended by the added entries (`landing-template-describe.md:22-23`). Nothing tells it to list those entries in the description. On an empty or no-pull-request landing, nobody reads the description before landing either.
Fix: drop that leg, or make listing the added entries a decision with an acceptance criterion.

F6. NIT · consistency. Location: D7, Bound.
Summary: "as batten's Run-Shed step already is" miscites. `internal/battencli/arm.go:514` says a wedged Run-Shed step is ended "only by the operator's pause or by ending the detached loop job", not by `lym batten cancel` or the row's own halts.
Fix: state D7's own bound without the comparison.

**Tool calls**
1. Grepped `logger.Debug` in the `Beam-Await` and `PR-Review` waits, fsnotify use, and `git show 5f444b56`: to learn whether the waits touch their trace, whether a file-watch helper exists, and to check the cited empty-landing commit.
2. Grepped `internal/loomcli/prcontinue.go` for the caller rule and arguments: to test D6's caller bound against the verb it is modelled on.
3. Grepped the Describe stencil location and `reedengine.ServerName`: to test D4's Describe leg and D9's server claim.
4. Listed the stencil directories: to find the Describe stencil.
5. Grepped `landing-template-describe.md` for what it reads of the record: D4's bound.
6. Grepped `internal/reedengine/bindings.go` for the re-pin rule: D9's re-pin claim.
