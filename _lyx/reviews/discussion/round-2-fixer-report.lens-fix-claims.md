```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that every bound, behaviour and code fact the fix added holds at HEAD.

**Finding 1: MEDIUM**
- **Changed text:** `decision-record.md` D4, new Windows paragraph: "a process started without the breakaway flag stays in its job, so every other descendant of a step still dies with it."
- **Summary:** The claim is too broad. One existing spawner already asks for breakaway, and the new `JOB_OBJECT_LIMIT_BREAKAWAY_OK` on the step and loop jobs changes what happens to it.
- **Evidence:**
  - `internal/shedverbs/loopwait.go:158-161`: `spawnLoop` starts a loop with `proc.DetachBreakaway`. When breakaway is refused it falls back to `proc.Detach` and logs that the loop "lives only as long as the shell's job".
  - Grep shows it is the only `DetachBreakaway` caller outside `internal/proc`.
  - Today that breakaway is refused under a step's tied job, because no job lyx creates sets `BREAKAWAY_OK` (`internal/proc/namedjob_windows.go:26` sets only `KILL_ON_JOB_CLOSE`).
  - After the change, a loop spawned from inside a step would leave the step's job and outlive it. So "every other descendant still dies with it" holds only for descendants that don't ask for breakaway.
  - I did not confirm whether any step currently reaches `spawnLoop`. The sentence should name the exception, or the plan should check it.

**Claims that hold:**
- **Finalize reroute (D2):** the loom reroute goes through `Parent-Align` to the Code perch. Evidence: `loom-recipe.yaml`, Finalize has `on_reroute: Parent-Align`, and `Parent-Align` has `on_done: Code-Bouncer`.
- **Watchdog wiring:** `agentActivityFor` feeds the step watchdog for every armed recipe, darn included, through `arm.go:334`. `step_idle_timeout_min` is in `internal/loomengine/template.yaml:44`.
- **F15 note:** darn uses `c.env`, assigned at `wiring.go:549`.
- **Adopt teardown:** the adopt ends the old pair through `pairteardown` (`internal/adoptcli/deps.go:39`).
- **Spawn budgets:** `tackler_max_spawns` and `arbiter_max_spawns` are counted per run (`internal/sheddriver/template.yaml:4,7`), and a spent budget hands the stop back (`sheddriver/doc.go:87`).
- **Hand-back:** `arbiterstop.HandsBack` keys on `DecisionHaltPrefix`.
- **Plan reviewers:** their tool-use rules say "build, run, test what you review" (`burlerengine/prompt.go:335`).
- **`spawnLoop` fallback:** matches the D4 text.

Tool calls I made:
1. Grepped the loom recipe for the Finalize and `Parent-Align` edges, which my context lacked.
2. Grepped `wiring.go`, the `agentActivityFor` callers and the `step_idle` template key, to check that the watchdog and env claims reach darn.
3. Grepped for loop-job teardown and adopt files, to check the adopt's end of the old pair.
4. Grepped `adoptcli/deps.go` and the `c.env` literal, to confirm the adopt goes through `pairteardown` and to find where the env is assigned.
5. Grepped for `DetachBreakaway` callers, to test the "every other descendant" claim.
