# Decision record — twill-discusser

## Goal

Add `twill`, a third Discussion-Write producer beside `single` and `seats`, selected with `discussion_producer: twill` in `loom.yaml`.
The writer explores and settles the design on its own, then forks itself once per lens before it writes; each fork waits, keeping its own prompt cache warm, until the written record is ready, reviews it through its lens as an outsider and returns findings; the writer takes them in and finalises.
The forks run on the writer's model and read the writer's exploration from cache instead of re-reading the code.

## Scope

In:

- The `twill` value of `discussion_producer`, its config validation and template comment.
- The twill Discussion spec: the single writer's spec with fork subagents authorized and a twill block rendered into the discussion template.
- A reusable twill directive stencil and the small package that renders it: the fork-wait-review procedure, with the reviewed files, the ready marker, the rubric to read and the lens list as markers.
- A `twill-discussion` fan and two new lenses (`premises`, `board-fit`) in the embedded burler lens library.
- `lyx shuttle await <path>... [--cap]`, the general keep-cache-warm wait on one or more files, usable by any agent; `lyx burler await-review` keeps its name, arguments and output and shares its poll implementation.
- Docs and stencil claims for all of the above.

Out:

- The bench comparing `twill` with solo `opus[medium]` and `opus[high]`: it runs after landing, from the prime, as its own job.
- Applying the twill directive to Plan-Write (board note "Later: Plan"): this task only keeps the directive and the wait verb free of Discussion-specific text.
- Using the wait verb for parent review (`parent-review-cold-cache`) or a seats chair (`burler-multi-llm`): those tasks adopt it; this task builds it general enough for them.
- Any change to the shed engine, the seat engine, `seats` or `single` behaviour.

## Decisions

### D1 — `twill` is a spec variant of the single writer, not a new engine

`discussion_producer` accepts `single`, `seats` and `twill`; the load error for any other value names all three.
With `twill`, the Discussion-Write row keeps the single writer's engine (shuttle through `DiscussionSpec`); `applyDiscussionProducer` and `DiscussionSeats` are untouched, and `DiscussionSpec` renders the twill block and sets `ForkSubagents` when the producer is `twill`.
The row's name, gates, routing, output files, timeout, role, segment colour and skills are those of `single`.

Rationale: to shed, `twill` is a single-LLM producer, as the board entry states; the forks live inside the writer's Claude Code session, so the row, the resume identity and the gate list need not change.

### D2 — The twill block is an optional marker in the existing discussion template

`loom-template-discussion` gains two optional markers, each rendering nothing for `single`:

- one placed between Step 4 (how to get answers) and Step 5 (write the two files), rendering the twill directive for `twill`;
- one directly below the "What you may write" file list, rendering for `twill` a write-scope paragraph: the ready-marker path is a third file the writer creates, the writer also creates the marker's parent directory, and both the lead-in's write scope ("the two named in Step 5") and the do-not-repair rule's "those two files" include the marker.
  The sentence that leaves committing "your two files" to the loop owner stays about the two discussion files only, since the marker is untracked state (D7) that nobody commits.

The section's own sentences stay as they are, so `single` renders them unchanged and the twill paragraph is the one place that widens them.
The write-scope paragraph is a second host-neutral stencil beside the directive (D3), with the marker path as its marker, rendered by the same package.
The directive points at the host template's Step 5 and Step 6 for the record format and the self-check, through D3's markers; it restates neither.
The writer's write scope grows by the ready marker and its parent directory, and nothing else.

Rationale: a separate full twill template would duplicate the decision-record format contract, which `PATTERN-producer-pointer-rule` forbids; optional markers keep `single`'s rendered prompt byte-identical.

### D3 — A reusable twill directive stencil, rendered by its own package

A new stencil (for example `contracts/stencils/twill/twill-directive.md`) holds the fork-wait-review procedure.
Its markers carry everything host-specific: the files the forks review, the ready-marker path, the review rubric the writer reads before spawning, the lens list, where the writer records each finding's disposition, the host steps that hold the output format and the self-check (Step 5 and Step 6 for Discussion), and the self-check command (`lyx loom validate-discussion` for Discussion).
A sibling write-scope stencil holds D2's paragraph, with the ready-marker path as its only marker.
A small package renders both from a told stencils directory at call time, the way `internal/editdirective` and `internal/parentdirective` render theirs.
It names no Discussion file, section or verb in its own text, so Plan-Write can render it later with its own values.

Rationale: the board entry asks that the fork-wait-review part stay reusable for Plan; `PATTERN-stencil-ownership` requires call-time reads.

### D4 — The writer's procedure

The rendered block tells the writer, in order, with the Discussion values in parentheses filled from D3's markers:

1. When the design is settled and before writing any output file, read the review rubric the directive names (`loom-rubric-discussion-review`), so every fork inherits it in the cached prefix.
2. Spawn every lens fork in one message through the Agent tool with `subagent_type: "fork"`, one per lens, never passing a `name`.
   Each fork's prompt is the fork boilerplate (D5) plus its lens text, and nothing else: no design hint, no direction to read code.
3. Write the reviewed files per the host's format step (Step 5) and run the host's self-check command (`lyx loom validate-discussion`) per its self-check step (Step 6) until it passes.
4. Write the ready marker.
5. Wait for every fork to return; ending a turn while forks are outstanding is allowed.
6. Judge every finding with the same skepticism, merge duplicates across lenses, and fix the record or reject the finding with a one-line reason.
7. Record the lens review where the directive names (the last turn of the support log's `## Interview`): one line per finding, its lens, and fixed or rejected with the reason; a fork that returned no report is named there too.
8. Re-run the self-check until it passes, then end the turn.

There is one fork pass per writer spawn: a gate re-prompt, a parent-review reject or a later turn of the same session never spawns forks again.
A writer whose session was relaunched after its process died (reed's resume, which types no prompt) has lost every fork it spawned, since forks live inside the dead process, and never spawns again.
On its next turn it resumes at the first of steps 3 to 8 it had not finished, with the findings it already holds, and step 7 records that the lens review was cut short: each fork that had not returned as "no report", or, when it died before step 2 finished, that no forks were spawned.
Its next turn is whatever next reaches the session, such as a gate re-prompt or a message from its parent; when none does, the run's existing timeout ends the run, and twill adds no prompt of its own.

Rationale: spawning after the design is settled gives each fork the whole exploration, including reads made while settling it, while no fork has written a line of the record; the marker after the self-check means forks never review a partial or malformed record.

### D5 — The fork's procedure, with its bounds

The fork boilerplate tells each fork:

- Its first and only action until the marker exists is `lyx shuttle await <marker>` with the Bash tool's `timeout` at 300000; on `ready: false` it re-runs the same command in a turn that does nothing else.
  It reads nothing, judges nothing and writes no note before `ready: true`.
- It gives up after the wait has reported `ready: false` 15 times in a row (about an hour at the default cap) and returns a one-line "no record to review" report.
- After `ready: true` it reads the decision record and the support log, then reviews the record through its lens.
- Its context already holds the writer's exploration and the rubric: it never re-reads what its context shows, and it makes at most 6 further tool calls, each for something its lens needs that the context lacks, such as a caller or spawned process one hop past what the record cites.
  Its report ends with one line per such call: what it read and why its context lacked it.
- It is read-only: no Write, Edit or delete, no git command, no Agent call.
- It returns its findings only as its final message, each with a severity (`BLOCKING`, `MEDIUM` or `LOW`), the record section or decision it concerns, a one-line summary and the evidence.

Bound: a fork costs at most one cached wait turn per cap while it waits, at most 15 such turns, and at most 6 exploratory tool calls once the record is ready.

Rationale: the capped foreground wait is the board entry's cache mechanism, the same one the Burler fixer uses; the give-up count and the tool budget are the fan-lab lessons (`fan-master-preread`) applied, so a writer that never writes the marker, or a fork that explores, cannot run up cost to the 480-minute run deadline.

### D6 — Fork subagents are authorized on the twill spec only

The twill spec sets `ForkSubagents`, so the Agent tool accepts `subagent_type: "fork"` and refuses every other type, and the fork-context `lyx webster` guard is installed; the `single` and `seats` specs keep today's full Agent deny.

Bound: while the shuttle config's `claude_deny_agent_tool` is true (the template default), the edge admits fork subagents only, refusing every other subagent type; with it false no Agent hook is installed, so the writer and its forks may spawn any subagent type, as `single` may under the same setting, while the webster fork guard stays.
The hook admits a fork spawn from any session in the run, so a fork spawning a further fork is refused only by the fork's stencil (D5), not by the hook.
Forks inherit every other standing deny of the writer's session (python, raw go, AskUserQuestion in autonomous mode).
A fork's read-only rule and its no-Agent-call rule are stencil-level only, as is the writer's own write scope.
The downstream gates (`validate-discussion`, parent review, Discussion-Review) read the two discussion files only: they bound what lands in the record, and nothing bounds a fork's write outside it beyond the stencil.
No Go-side fork audit is added; shuttle's `ForkAudit` for a done fork-mode run still records each fork's writes, Agent calls and Bash commands, read by no twill policy.

Rationale: the existing fork mode is exactly the needed permission; an audit like Burler's cluster round adds Go for a risk the downstream gates already bound.

### D7 — The ready marker is ephemeral and never stale

The ready marker lives under `.lyx` at the subpath mirroring `_lyx/discussion/`, its path built through `internal/lyxdirs`.
A marker left by an earlier writer spawn never wakes a fork of a later spawn: either the path is unique per fresh spawn or Go removes it before a fresh spawn; an attach to a live writer, or a relaunch of a dead one (D4), keeps the marker, since a relaunched writer spawns no forks to wake.
The marker is not an output file, so shuttle's completion check and the gate list are unchanged.

Rationale: `PATTERN-durable-vs-ephemeral-state` and `PATTERN-lyxdirs-single-declarer`; a stale marker would send forks to review an old record without waiting.

### D8 — Lenses come from the burler lens library, fan `twill-discussion`

The embedded burler template gains the fan `twill-discussion`: `premises`, `goal-scope`, `board-fit`, `pattern-fit`, `scenarios`, `leanness`.
It gains two lenses, phrased as emphasis, never exclusion, like every lens:

- `premises`: every claim the record makes about existing code, config or behaviour matches what the code does, including one hop past what it cites.
- `board-fit`: the record against every other board entry: planned work it duplicates or contradicts, and a dependency it leaves unnamed.

The twill spec resolves the fixed fan name `twill-discussion` through `burlerengine.ResolveFan` against the worktree's burler config, so an operator who redefines the fan in `burler.yaml` changes the lenses with no loom key.
A resolution failure fails the spawn with an error naming the fan and `burler.yaml`.
The forks run on the `discussion` model; there is no lens or model key of twill's own.

Rationale: one lens library, already operator-editable with template fallback; `scenarios` and `leanness` cite the Discussion-Review rubric, which the writer reads before spawning (D4), so they apply unchanged.

### D9 — `lyx shuttle await`, the general keep-cache-warm wait

`lyx shuttle await` is the one capped wait any agent uses to keep its prompt cache warm while it waits on a file.
Its stencil-facing contract names no Twill, Discussion or loom file.

- `lyx shuttle await <path>... [--cap <duration>]` takes one or more paths and returns as soon as any of them exists, printing ok with `ready: true`, the absolute path that was met and the absolute paths it watched.
  When the cap elapses first it prints ok with `ready: false`, the watched paths and a "call this command again" message; it exits 0 either way.
- A marker is a path like any other: the condition is "this file appears", whoever writes it.
- The default cap is 250 s, under the 5-minute prompt-cache lifetime; the caller runs it with a Bash tool timeout above the cap (300000 ms), and re-runs it in a turn that does nothing else, which reads its context from cache and resets the clock.
- A relative path resolves against the seam cwd through `lyxcwd.CwdFrom`; a stat error other than not-exist, or a cancelled context, is a JSON error with exit 1.
- It is read-only, carries the role audience, and skips the shuttle group's geometry, config and engine resolution, so it needs no hub or git repository and runs the same in a fork, a chair or a writer.
- The shared poll stats the watched paths once a second.
  It meets `PATTERN-daemon-wakeups` whether or not a capped call over a minute is in that rule's scope: it wakes at the one-second floor, a stat spawns no process and captures no pane so the backoff clause does not apply, and it uses no `internal/fswatch` watcher, since a watched path's parent directory may not exist yet.
- `lyx burler await-review` keeps its name, single argument, default cap and output keys and calls the same poll implementation, so its poll moves from 200 ms to the one-second floor.

An incoming SendMessage is not a condition the verb watches: lyx cannot see Claude Code's inbox, and a queued message drains at the waiter's next tool round, so a message that arrives during a wait is taken in at most one cap late.
A protocol that needs the waiter to wake on a message at once has the sender also write a marker the waiter watches.

Bound: the verb only stats the told paths; each call ends by its cap.

Rationale: shuttle runs every agent, so a wait every role can use belongs there; watching several paths lets one call cover "the reply or the stop marker" without a verb per caller; keeping `await-review` means a Burler fixer spawned before a deploy still finds its verb after the prime runs `update-plugins.sh` mid-run.

## Constraints

- `PATTERN-producer-pointer-rule`: the twill directive points at the host template's format and self-check and never paraphrases them.
- `PATTERN-stencil-ownership`: the directive, the discussion template and the lens library are read at call time from the told stencils directory.
- `PATTERN-parent-directive`, `PATTERN-edit-directive`, `PATTERN-role-skills-typed`, `PATTERN-spawn-color`: the twill directive is not a top-level stencil; it asks no agent to load a skill, set a colour or ask the operator.
- `PATTERN-no-denied-recovery`: no step in the directive names a command the writer's or a fork's settings deny.
- `PATTERN-cli-cobra`, `PATTERN-cwd-resolution`: the await verb is a cobra verb in the shuttle subtree with `Short`, audience and JSON errors, and resolves a relative path only through `lyxcwd`.
- `PATTERN-durable-vs-ephemeral-state`, `PATTERN-lyxdirs-single-declarer`: the marker is under `.lyx`, spelled only through `internal/lyxdirs`.
- `PATTERN-completion-signal`: output files stay the two discussion files; nothing new decides "did this run finish".
- `PATTERN-daemon-wakeups`: the shared poll wakes at most once a second (D9); the loop without a deadline of its own is the waiter's turns, at most one per cap.
- `PATTERN-leaf-packages`: the directive renderer's imports fit the closed set the background names for directive packages.
- `PATTERN-refusal-way-forward`: the unknown-producer load error and the fan-resolution error each name the way forward.
- `PATTERN-test-speed`, `PATTERN-test-economy`: unit tier, injected clock for the wait, existing tables extended where one fits; new top-level tests stay within `cmd/lyx/testdata/test-budget.yaml` or the card lists it under `Edit:`.
- `PATTERN-documentation-lifecycle` and CLAUDE.md "Docs land in the same commit": `loomengine` and `shuttlecli` doc.go, the directive package's doc.go, the loom template comment, and `docs/overview.md` if its module table or execution stack names the Discussion producers.

## Auto-mode assumptions

- Forks are spawned after the design is settled (the board's "ready to write"), not right after exploration.
- The writer reads the Discussion-Review rubric before spawning, so the rubric-citing lenses apply unchanged.
- Fan composition: six lenses, `premises`, `goal-scope`, `board-fit`, `pattern-fit`, `scenarios`, `leanness`; no `generic` lens.
- The fan name is fixed; `burler.yaml` is the knob, with no new loom key.
- Fork give-up after 15 consecutive `ready: false` answers; fork tool budget 6.
- Finding dispositions go in the support log's `## Interview` as its last turn; `## Review rounds` stays the gate's.
- The wait verb lives in shuttle, named `await`; `await-review` stays as a thin verb over the shared poll.
- The wait's conditions are file paths, any one of which ends it; an incoming message is not watched, so a sender that needs an immediate wake also writes a marker.
- `twill` works the same in interactive and autonomous mode.

## Open risks

- The forks inherit the writer's reasoning up to the spawn, so they are outsiders to the text, not to the design; how much that blunts the review is what the bench measures.
- The writer waits idle while its forks review; a lens review longer than 5 minutes wakes the writer on a cold cache, and nothing here keeps it warm.
- Six forks may cost more than they find; Burler's fan raised mostly MEDIUM and LOW findings with many duplicates across lenses.
- Claude Code's fork and background-task behaviour is external: a change in how forks inherit context, report completion or surface as outstanding work in the session signals would break the wait without a code change here.
- If the single producer is later parked in favour of seats with no advisors (`discussion-multi-llm-bench`), the twill block must move with the template it is rendered into.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - marker-write-vs-host-write-scope (class consistency, severity MEDIUM, open in rounds 1, 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- `discussion_producer: twill` loads; an unknown value is refused with an error naming `single`, `seats` and `twill`.
- With `single`, the rendered Discussion prompt is unchanged and the spec keeps the full Agent deny; with `seats`, nothing changes.
- With `twill`, the spec sets fork subagents, keeps the row's output files, model, timeout and gates, and its prompt carries the twill block with every lens of the resolved fan, the ready-marker path, the rubric to read and the await command.
- With `twill`, the rendered prompt's "What you may write" section says the files the writer may create or modify are the two discussion files and the ready-marker path, with the marker's parent directory, and widens the lead-in's write scope and the do-not-repair rule's "those two files" to include the marker, while the commit sentence stays about the two discussion files.
- The unrendered twill directive stencil names no Discussion file, section, step or verb.
- A `burler.yaml` that redefines `twill-discussion` changes the rendered lenses; an unresolvable fan fails the spawn with an error naming the fan and `burler.yaml`.
- The twill directive's wording claims are pinned: fork subagent type without a name, one spawn message, wait before any lens work with a 300000 ms Bash timeout, re-run on `ready: false`, the give-up count, the tool budget with its end-of-report lines, read-only, findings only in the final message, marker only after the self-check, one fork pass per spawn, and a relaunched writer resuming at its first unfinished step and recording its lost or never-spawned forks without re-forking.
- `lyx shuttle await`: returns `ready: true` at once for an existing path, naming it; with several paths, returns when any one appears and names that one; returns `ready: false` with exit 0 when the cap elapses; resolves a relative path against the cwd; reports another stat error as a JSON error with exit 1; runs outside a git repository and outside a hub.
- `lyx burler await-review` keeps its behaviour and output.
- A marker from an earlier writer spawn does not satisfy the wait of a later spawn's forks.
- A gated Discussion-Write turn end with forks outstanding and both output files present is a wait, not an arrival (existing shuttle coverage may meet this).
- The new lenses resolve through `ResolveFan`, and the embedded template still decodes strictly.

## Notes for the plan writer

- Discussion producer selection: `internal/loomengine/config.go` (`DiscussionProducer*`), `internal/loomrecipe/loomrecipe.go` (`applyDiscussionProducer`), `internal/loomcli/wiring.go` (`DiscussionSeats`, `DiscussionSpec`).
- Prompt composition: `internal/loomengine/discussion.go` (`DiscussionSpec`) and `prompt.go` (`composePrompt`, `stencil.FillOptional`).
- Fork authorization: `shuttleengine.Spec.ForkSubagents` and its standing denies in `internal/shuttleengine/claudeengine/settings.go`; the Burler cluster round is the existing user.
- Fork boilerplate precedent: `clusterRulesBlock` in `internal/burlerengine/prompt.go`, and the variant D text in the `fan-master-preread` board entry.
- Wait precedent: `internal/burlercli/await.go`; its fixer-stencil claim lives in `contracts/stencils/claims_test.go`.
- Waiting-turn-end handling for forks: `expiredTurnEnd` in `internal/shuttleengine/wait.go`.
