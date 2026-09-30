# Support log — plan-write-respawn

## Interview

Autonomous session: no operator was present, so every turn below is exploration followed by a self-pick.

1. **Board entry.**
   Task 14, two friction items from run `operator-surface`: a respawned Plan-Write discards the finished plan, and the Agent-tool refusal is not stated in the prompt.
   The false `asking` that triggered the respawn is already fixed in `083e992cf`; this task covers any respawn.
2. **Where the plan goes on respawn.**
   `SingleLLMProducer.Call` builds the spec, probes for a live run, then runs the fresh-spawn preparation (Plan-Write's `NewPlanDirRotator`), then archives the spec's output files, then spawns.
   The rotator moves top-level `.md` files into `archive-<stamp>[-N]`.
   Plan-Write runs once per forward pass; the review segment fixes the plan through Plan-Burler, which never rotates.
   So Plan-Write only re-runs after its own halt, or on a restarted run.
3. **Why the rotation cannot simply be dropped.**
   `shuttleengine.Spec.OutputFiles` must not exist at spawn: the first turn end would classify Done on a stale `00-overview.md`.
   Decision: keep the rotation, point the prompt at its result (D1).
4. **Reuse without an agent?**
   Considered short-circuiting to Done when a finished plan passes the plan gate.
   Rejected (D3): the halt that caused the respawn may have invalidated the plan in a way the gate cannot see.
5. **How the prompt learns the archive path.**
   The spec is composed before the probe; the archive path exists only after the rotation, which must run after the probe.
   Attach matches on output files only, so amending the prompt after the probe is safe.
   Decision: widen the preparation seam so it can amend the prompt (D4), with the block text in a stencil (D5).
6. **What the block tells the agent.**
   Carry forward what still holds, re-verify glyphs with quarry, revise only what is wrong, leave the archive alone (D2).
7. **Where to announce the Agent deny.**
   The deny is installed by `claudeengine/settings.go` from `claude_deny_agent_tool`, in every run, with a fork-mode variant.
   A static sentence in the Plan template would lie if the config switched the deny off, and would leave every other spawned agent uninformed.
   `claude --help` (2.1.285) lists `--append-system-prompt` and states the appended prompt is recorded on the first request and replayed on resume.
   Decision: the engine announces each installed deny through `--append-system-prompt`, from the same inputs as the hooks, on launch and resume (D6).
8. **Which denies to announce.**
   Agent (both variants) and AskUserQuestion; not the webster fork guard, whose audience is forks and whose text would mislead the Master (D6).
9. **Quoting.**
   Notices ride a pane-shell command on every supported shell; the webster steer already carries a no-quote, no-backslash check (D7).

## Rejected alternatives

- **Skip the agent when a finished plan passes the gate.**
  A respawn follows a halt whose cause (an asking stop, an operator edit to the decision record) the gate cannot see; a stale plan would pass as Done.
- **Leave the plan in place and move only `00-overview.md`.**
  Leaves a half-rotated directory mixing old cards with new ones, which every downstream reader (gate, Plan-Bouncer, Batchifier) would take as one plan.
- **Static prompt text: "check the newest `archive-*` directory".**
  Needs no plumbing, but points at archives from unrelated older runs as readily as at this run's, and costs every first spawn a directory scan.
- **Re-invoke the spec source after the preparation, with `PlanSpec` detecting the new archive.**
  `PlanSpec` is a pure composer that stats nothing; making it stat the plan directory, and guess which archive is the fresh one, breaks that and still cannot know what the rotation moved.
- **An optional `{{.prior_plan}}` marker in `loom-template-plan.md`.**
  The value is unknown at compose time, which is when markers are filled.
- **A static "the Agent tool is refused" sentence in `loom-template-plan.md`.**
  Drifts from `claude_deny_agent_tool`, and fixes one template while every other spawned agent stays uninformed.
- **Prepend the notice to the prompt text in `claudeengine`.**
  Templates user-prompt content inside the transport; a system-prompt flag is provider vocabulary the engine already owns, the same way it owns `--settings`.
- **Implement the Someday "shuttle `Spec`: generic tools-restriction" item.**
  No new restriction is needed; the friction is that an existing restriction is silent.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Rotate or reuse in place on respawn?**
  Auto-pick: keep the rotation, point the prompt at the archive (D1).
  Resolved.
- **Q2 — Should a finished, gate-passing plan skip the agent?**
  Auto-pick: no, the agent always runs (D3).
  Resolved.
- **Q3 — How does the prompt learn the archive path?**
  Auto-pick: widen the fresh-spawn preparation seam to amend the prompt (D4).
  Resolved.
- **Q4 — Where does the block's prose live?**
  Auto-pick: a new loom stencil, appended to the prompt, plus one static line in the Plan template (D5).
  Resolved.
- **Q5 — Where is the Agent deny announced?**
  Auto-pick: in `claudeengine`, via `--append-system-prompt`, built from the same inputs as the hooks (D6).
  Resolved.
- **Q6 — Announce AskUserQuestion and the webster fork guard too?**
  Auto-pick: AskUserQuestion yes; webster fork guard no (D6).
  Resolved.
- **Q7 — Extend the fix to Discussion-Write and Describe respawns?**
  Auto-pick: no; the board task names Plan-Write only, and neither row's friction was reported.
  Resolved, out of scope.
- **Q8 — Carry the prior session's stop reason into the respawn prompt?**
  Auto-pick: no; not asked for, and the agent re-derives what matters by checking the plan against the decision record.
  Resolved, out of scope.
- **Q9 — Does the roadmap move?**
  Auto-pick: no; this is a fix to existing behaviour, not a planned item.
  Resolved.
