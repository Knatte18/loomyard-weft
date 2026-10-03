# Decision record — agent-self-reflect

## Goal

Every loom run's friction reaches GitHub as issues with no operator in the loop, under `lyx shed step` / `lyx loom step` as well as `lyx loom run`.
Agents and Go write friction down into the run's friction directory, one reflection agent files it through `lyx selfreport create`, and nothing deletes or archives a note before it has been reflected on.
Filtering of what was filed happens later, when issues are pulled back into the board, not before filing (operator, 2026-10-03).

## Scope

In:

- Friction-Reflect reflects under `step` as it does under `run`.
- A run that halts (`blocked` or `failed`) under either verb gets a Go-written halt note and a reflection, as `run` already does for `blocked`.
- Every refusal of a `lyx webster` verb writes a Go-authored friction note when the friction directory resolves.
- ly-drive writes its repair and re-step records into the run's friction directory, may write its own optional friction note, files nothing itself, and notifies the parent when a done run's reflection failed.
- `frictionengine.Reflect` becomes re-entrant: an interrupted or killed driving process never leads to a second agent filing the same notes.
- The reflection stencil leans toward filing: it drops only items that are not about lyx and merges duplicates.
- Docs and `CONSTRAINTS.md` updated in the same commit.

Out:

- Tier 1 anomaly filing (`detectAndFileAnomalies`) stays `run`-only.
- Per-agent end-of-session self-report sessions (a `mill-self-report` analogue for Merriam or the driver).
- Deduplication against already-open GitHub issues.
- The board intake that pulls issues back and filters them.
- Reflection for a run torn down without reaching `done` or a halt (manual `lyx fabric remove`).
- Batten's own recipe: it has no friction directory and gets none.

## Decisions

### D1 — Friction-Reflect reflects under step

Decision: `reflectFrictionRow` drops the `armedVerb == "run"` gate; whenever the friction directory is non-empty it calls `reflectFriction(true)`, under `step` as under `run`.
The row still always returns Done, and done still persists only after the reflection returns, so batten's Run-Shed sees the child as running until the reflection ends and tears nothing down before it.
Rationale: every driven run uses `step`, so the gate meant no driven run ever reflected.
The original reason for the gate, that the reflection agent files public issues itself, is overruled by the operator's decision that filing needs no approval.
Bound: the reflection blocks the step for at most `friction_timeout_min` (loom config); it files only to the hardcoded `Knatte18/loomyard` repo through `lyx selfreport create`, and what an issue body may carry is bounded by D6's content rule.

### D2 — Halts reflect under step, and Go writes a halt note first

Decision: when a `step` or `run` invocation leaves loom's run `blocked` or `failed`, loomcli first writes one Go-authored friction note (producer, state, the envelope's reason, history position, trace file path), then runs the non-waiting reflection `reflectFriction(false)` exactly as `run`'s blocked path does today.
Under `step` this happens in the step verb's own process before the envelope is written, and the step envelope carries the same `friction` status key `run`'s envelope carries.
`awaiting` and `paused` write no note and reflect nothing.
Rationale: a halt is the moment a human would otherwise investigate and file by hand (#350 and #352 were both filed by the hub at a Webster block); reflecting at the halt files them then, and archiving the reflected notes means the terminal reflection never refiles them.
A Go-written note makes halt friction mechanical rather than dependent on an agent remembering.
Bound: the halt step's envelope is delayed by at most `friction_timeout_min`; a held reflection lock skips the reflection rather than waiting (today's blocked-path behaviour).

### D3 — Webster verb refusals write a friction note

Decision: every refusal envelope from a `lyx webster` verb except `validate` writes one Go-authored friction note through `friction.NotePath`, holding the verb, its arguments, the refusal text, and its class where the refusal carries one.
It is written only when webstercli's tolerantly resolved friction directory is non-empty (hub mode with Tier 2 on), and a write failure only logs, never changes the verb's exit or envelope.
Rationale: Merriam meets refusals from `rebaseline`, `begin-batch` and `recover-batch` that today leave no record; writing the note in Go, at the one place the refusal is produced, needs no agent cooperation and follows the existing `writeTriageFrictionNote` precedent.
`validate` is a lint an agent iterates against, so its findings are expected and not friction.
Bound: one note per refusal; the note is a local file in the friction directory and files nothing by itself, and D6's content rule bounds what of its arguments and refusal text an issue may quote.

### D4 — ly-drive's friction goes into the friction directory; ly-drive files nothing

Decision: in ly-drive's skill text:

- Repair records and automatic re-step records are written into the envelope's `friction_dir` when it is non-empty, and under `<scratch_dir>/repairs/` only when it is empty.
- The driver may write one optional friction note of its own into `friction_dir`, under the same "only when something went wrong" rule the other agents' directives use.
- The `## Self-report` section is replaced: the driver drafts and files no issue in any mode; the run's reflection files.
- At a `done` stop whose envelope reports `friction: failed`, the driver notifies the parent like an escalation, naming the run-id and the stop report.

The skill stays free of row and recipe names: it branches only on `friction_dir` and the `friction` envelope key.
Rationale: one filer per run means no duplicate issues between the driver and the reflection, and records in the friction directory are archived with the notes after reflection, so nothing is filed twice.
Repair records are already declared friction data by the skill.
Bound: the parent notification on `friction: failed` is the only new notification; a done stop otherwise still sends nothing.

### D5 — Reflect is re-entrant across a dead driving process

Decision: `frictionengine.Reflect` records the set of notes a reflection covers before spawning its agent, in a non-`.md` record file in the friction directory beside the report, so the note scan never reads it.
On entry it first probes for a live reflection agent through shuttle's attach seam and waits on it instead of spawning; when no agent is live but a report from a finished reflection exists, it archives the notes that reflection covered instead of spawning again.
A finished reflection is one whose report exists; under D6 the agent writes the report only after its filing succeeded.
Every archive, on a clean return and on the no-spawn path alike, moves only the covered notes, the record and the report; a note written after the record stays in the friction directory for the next reflection.
A report with no record is stale and deleted as today; a record with no report and no live agent is discarded, and its notes are reflected again.
The probe runs inside Reflect, under loomcli's friction lock: D2's skip on a held lock and the terminal row's wait on it are unchanged, and the lock is advisory, so a killed process leaves none behind.
Friction-Reflect's interrupt policy stays `reinvoke`, now justified by the attach probe rather than by "step spawns nothing".
Rationale: under `step` a killed step process (a harness job limit, a crash) leaves the reflection agent running in its tmux strand with Go's archive step never run; without re-entrancy the next invocation would file every note a second time.
Bound: a re-invocation spawns at most one new agent, only over notes outside every finished reflection's covered set; it archives without spawning only on a report, which proves filing succeeded.
A reflection killed mid-filing leaves no report, so its notes are reflected again and the issues it filed before dying are filed a second time.

### D6 — The reflection files every distinct lyx problem

Decision: the reflection stencil (`friction-template-reflection.md`) tells the agent to file every distinct problem in lyx tooling the notes describe, to merge notes describing the same problem into one issue, and to drop only notes about the task's own code or describing nothing wrong.
It names the note kinds it will see: agent notes, Go halt and refusal notes, and driver repair records.
Each issue body ends with a provenance line naming the task slug and the note file names it came from.
An issue body describes the lyx problem and may quote lyx's own output, verb names and repo-relative lyx paths; it never quotes the task repository's source, diffs or plan text, credentials, tokens, environment values or absolute home paths.
The report becomes a success marker: the agent writes it only after every issue it decided to file was created, or after deciding nothing is worth filing.
On a failed `lyx selfreport create` it ends its turn without writing the report, so shuttle classifies the run `asking`, Reflect returns `failed`, and the notes stay unarchived.
Rationale: the operator moved filtering to intake; the reflection's job is grouping and provenance, not triage.
Go never parses the report, so its existence is the only signal that filing succeeded; a report written after a failed filing would archive notes that never reached GitHub, with status `reflected` and no D4 alert.
Bound: filing stays `lyx selfreport create` only, never `gh`; `Knatte18/loomyard` is public, so the content rule above is the only check on what a note's text can publish.

### D7 — A new invariant in CONSTRAINTS.md

Decision: add a Friction Capture invariant: a loom halt (`blocked` or `failed`) and every `lyx webster` refusal but `validate`'s write a Go-authored friction note when Tier 2 is on, and no code path archives or deletes a friction note before a reflection has covered it.
Rationale: both are cross-cutting rules a later change could silently break; per CLAUDE.md a new invariant goes into `CONSTRAINTS.md` in the same commit.

## Constraints

- Filing goes through `lyx selfreport create` only; no `gh` writes anywhere in this task.
- Every agent lyx spawns runs as an interactive tmux session through reed and shuttle; the reflection agent keeps using shuttle.
- The reflection step never changes a run's outcome: Friction-Reflect always returns Done, a halt stays a halt, and every runtime failure below Deps validation returns a nil error.
- Batten's recipe and teardown are unchanged: ordering rests on done persisting only after Friction-Reflect returns (D1) and on halt reflections running inside the halting step (D2).
- Told-Geometry: shedverbs takes no new `lyxcwd` dependency; the friction directory stays told through `Spec`.
- Friction Leaf Invariant: `internal/friction` gains no feature-package import; note writing for halts and refusals lives in loomcli and webster code, using `friction.NotePath`.
- The step envelope's key set is closed by doc comment and test; adding `friction` to step envelopes updates both.
- ly-drive carries no row or recipe names.
- Markdown in semantic line breaks.
- Docs in the same commit: `docs/overview.md` (frictionengine and Friction-Reflect lines, the `awaiting` line that says it triggers no reflection), `CONSTRAINTS.md`, the `internal/friction` and `internal/frictionengine` package docs, the loom recipe's Friction-Reflect comment, and `InterruptPolicies`' doc comment.

## Auto-mode assumptions

- Halt notes and reflection cover `failed` as well as `blocked`, under both verbs; `run` today reflects on `blocked` only.
- Webster refusal notes cover every webster verb except `validate`, whoever calls the verb (Merriam, a fork, the driver or the hub).
- Drive reports are not a reflection source: their friction content is covered by halt notes and repair records in the friction directory.
- No dedupe against open GitHub issues; the operator's statement that filtering happens at intake is taken as deciding this.
- No new GitHub label; provenance travels in the issue body.
- ly-drive needs no text about the reflection's wait: it already waits on the step job's exit with no time limit, and D5 covers a killed job.

## Open risks

- A failed terminal reflection (timeout, a failed `lyx selfreport create` under D6's report rule) still lets `done` persist; its notes survive only in the run records and the `archive/` tag, and refiling them takes a person, alerted by D4's parent notification.
- A reflection that fails partway through filing, or is killed mid-filing, leaves no report, so the next reflection refiles the issues it had already created.
- A run torn down by hand without reaching a halt or `done` loses its unreflected notes to the archive tag only.
- The same lyx bug hit by several runs is filed once per run until intake filtering exists.
- A long reflection at a halt delays the driver's hand-back by up to `friction_timeout_min`.
- Webster refusals that an agent hits and resolves routinely (a transient lock, a rebaseline it is told to run) produce notes the reflection must recognise as noise; D6 lets it drop only non-lyx items, so some low-value issues may be filed.

## Acceptance criteria

- Under `step`, a run reaching Friction-Reflect with notes in the friction directory spawns the reflection agent through shuttle and archives the notes on a clean return; under `step` with Tier 2 off it spawns nothing.
- A `step` that leaves the run `blocked` writes a halt note naming the producer and reason, runs the reflection before the envelope is written, and the envelope carries the `friction` status; the same holds for `failed`, and for both states under `run`.
- An `awaiting` or `paused` result writes no halt note and spawns no reflection.
- A `begin-batch` refusal with `plan_drifted`, a `rebaseline` refusal and a `recover-batch` refusal each write one friction note holding the verb and the refusal text; a `validate` failure writes none; an empty friction directory writes none and the verb's envelope is unchanged.
- A note-write failure for a halt or refusal leaves the verb's exit code and envelope unchanged.
- Reflect re-entrancy: with a live reflection agent, a second Reflect attaches and spawns nothing; with a finished report and unarchived covered notes, a second Reflect archives them and spawns nothing; a note added after that reflection started is reflected by the next spawn; a covered-notes record with no report and no live agent is discarded and its notes are reflected anew.
- A note written while a reflection runs is left in the friction directory by that reflection's clean-return archive.
- A reflection agent that ends its turn without a report yields `failed`, archives nothing, and a done stop's envelope carries `friction: failed`.
- The terminal reflection after a halt reflection covers only notes written after the halt reflection archived its set.
- Friction-Reflect's interrupt policy is `reinvoke` and its doc comment states the attach premise.
- ly-drive's skill text: repair records go to `friction_dir` when non-empty; no `lyx selfreport create` instruction remains; a `done` stop with `friction: failed` notifies the parent; no row or recipe name appears.
- The reflection stencil instructs filing every distinct lyx problem, grouping duplicates, and ending each issue body with the provenance line; it states D6's content rule and tells the agent to write the report only after filing succeeded.
- `CONSTRAINTS.md` holds the Friction Capture invariant; the step envelope key-set doc and test include `friction`.
- Done-when scenario, driven by a test with a fake shuttle and fake issue filer: a stepped run whose Webster row hits a `plan_drifted` refusal and blocks ends that step with a reflection whose spawn sees both the refusal note and the halt note.

## Notes for the plan writer

- Gate to remove: `reflectFrictionRow` in `internal/loomcli/run.go`; blocked-path call in `loomPostRun` (`internal/loomcli/arm.go`); step has `PreStep`/`PostStep` hooks in `internal/shedverbs/spec.go` but no post-halt hook today, and `PostStep` runs on the success path only.
- Precedent for a Go-written note: `writeTriageFrictionNote` in `internal/websterengine/triage.go`; webstercli resolves `frictionDir` in `internal/webstercli/wiring.go`.
- Attach seam: `internal/shedadapters/doc.go`, "Every spawning adapter probes for a live agent first".
- `deleteStaleReport` in `internal/frictionengine/reflect.go` assumes a leftover report is stale; D5 changes that meaning.
- `#274` is closed; the board entry's note `batten-llm-friction` is folded into this task.
