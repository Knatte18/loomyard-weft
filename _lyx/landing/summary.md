# Load all of a role's skills in one turn, and skip unknown ones at once

A spawned role, and the orch after each reload, now loads its whole skill list in one model turn instead of one `/<skill>` slash-command turn per skill.
A skill the session does not know is skipped when that turn ends, with a Warn naming it, so it no longer stalls a launch or a reload.

Closes #394

## What changes

- `claudeengine` types one plain message, with no leading `/`, that asks the model to load every listed skill through the Skill tool in list order and reply only `ok`.
  A Skill-tool load registers the skill as invoked, so Claude Code re-injects it after a compaction.
- At the load turn's end the engine reads the transcript the Stop payload names and classifies each requested skill, in the new `claudeengine/skillload.go`:
  - **loaded**: a main-chain Skill call naming it, after the load message, with a successful result.
  - **unknown**: absent from the transcript's skill listing, or its Skill call returned an error.
    It is skipped at once with a Warn (cause `unknown`) and never retried.
  - **missing**: neither of the above.
    It is retried once in a message naming only the missing skills, then skipped with a Warn (cause `not loaded`).
- `shuttleengine`: `SkillLoader` is list-shaped, and `Run`'s launch step spends one load turn, plus at most one retry turn, before the prompt pointer.
  `Runner` exposes `LoadSkills` and `ClassifySkillLoad` for the orch watcher.
- `orchengine`: the reload sequence is a skills step, an optional retry step, then the pointer step.
  `ReloadRetry` joins the persisted state, so a restarted watcher neither re-sends a confirmed step, nor retries twice, nor restarts a step's timeout.
  `internal/orchcli`'s adapter follows.
- The pane-text `SkillUnknown` classifier and the per-skill `/<skill>` load are removed.
  Its needles never matched Claude Code's `Unknown command: /<name>` notice, which was the cause of #394's 600 s stall.
- Docs are updated in `shuttleengine/doc.go`, `claudeengine/doc.go` and `orchengine/doc.go` ("The reload sequence"), and in the `Spec.Skills` and `Spec.SkillLoadTimeout` comments.

## Design decisions to check

- The message text lives in `claudeengine`, beside the `/clear` and `/compact` sequences.
  `shuttleengine` and `orchengine` pass only provider-neutral skill names, per `PATTERN-shuttle-provider-seam`.
  It is a fixed command line, not a stencil, so `PATTERN-role-skills-typed` is unaffected.
- An unreadable transcript, a missing `transcript_path`, no load message, or no skill listing degrades to one `skill load unverified` Warn listing the skills.
  The turn end then confirms the load with no retry, on the retry turn as on the first.
  Such a turn can no longer detect an unknown skill, but it never stalls.
- `Spec.SkillLoadTimeout` keeps its name and default and now bounds one load turn.
  A turn that times out skips every skill it asked for and is not retried.
- Shuttle's events offset ends past every load turn end, retry included, so `Wait` never reads a load turn as the run asking.
- A persisted `reload_step` from the old per-skill indexing that falls past the skills step reads as the pointer step.
  The worst case is one reload that skips the remaining skills of that cycle.
- No extra reload is added for a Webster master or another spawned role after compaction.
  This relies on Claude Code re-injecting Skill-tool loads, which is assumed and not verified.
- In the `internal/loomcli` tests, `integration_driverbootstrap_test.go` and `smoke_driverstrand_test.go`, each shell stub now answers the first typed line by appending a Stop event with no `transcript_path`.
  The load then confirms as unverified at once.
  Only the skill-load answer and its comment changed in each file.

## To check or run by hand

- Launch a role whose skill list contains a nonexistent skill.
  Confirm it is skipped at the first turn end with a Warn and no retry.
- Run `lyx orch cycle` and confirm the reload takes one skill turn.
- The `tmux`-tier tests in `integration_driverbootstrap_test.go` and `smoke_driverstrand_test.go` have no recorded run.
  Run them and confirm `lyx loom start` returns inside the 30 s bound.
- Confirm against a live session that a Skill-tool load survives a compaction, which the no-extra-reload decision above assumes.
- `gofmt -l` flags five files this change did not touch.
