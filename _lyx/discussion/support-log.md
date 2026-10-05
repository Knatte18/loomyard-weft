# Support log — selfreport-issues-say-nothing

## Interview

The session ran autonomously; every turn below is a self-answered question after reading the code.

1. Is this the right thing to build?
   Yes.
   Issues #370 and #376 confirm the board entry: the body renders blank state, producer and error, and "no history entries", because the anomaly only carries the fields its trigger reads.
   The Go filer cannot explain an event; the reflection agent can.
2. Which kinds still exist?
   `a0dae77c3` removed `recurring-finding`; four remain: `crash-resume` and the three halt kinds.
3. How do anomalies reach `Friction-Reflect`?
   Through the friction directory, the reflection's existing input.
   The three halt kinds map one-to-one onto the existing `loom-halt` note's cases (blocked with the budget prefix, blocked otherwise, failed), so the halt note gains the kind and history rows instead of a second note.
   Crash-resume gets its own note.
4. When is the crash-resume note written?
   At the entry observation.
   Deferring to post-run only served the filing pass, and it forced the cancelled-context exception.
5. How can Go tell a deliberate resume from a crash?
   Not from disk: a migration and a crash leave identical state.
   Every resume, whether by operator, orch or batten, goes through `lyx loom start`, so start vouches for the next observation, using the same one-shot, state-matched voucher the step handoff already uses.
6. What gates note writing now that `selfreport` is gone?
   Tier 2's own on/off: an empty friction directory means off.
7. Labels?
   The agent picks through `lyx selfreport create --label`; the stencil notes that a process signal is not a defect.
8. How do we keep this from regressing?
   A test-enforced PATTERN entry: only `selfreportcli` calls `CreateIssue`.

## Rejected alternatives

- Keep Go filing but enrich the issue body with full status and history: the board entry rules Go filing out, and a richer body still cannot say why the event is abnormal or what to do.
- Pass anomalies to the reflection as a structured file or a new stencil marker: this adds a second input channel beside the friction directory, plus archive and re-entrancy handling `frictionengine` already provides for notes.
- A separate note per halt kind beside `loom-halt`: this duplicates the halt note for the same event.
- Drop crash-resume detection entirely: a bare `run`/`step` that finds a dead driver's run, such as a step killed mid-producer, is a real lyx-relevant event worth a note.
- Detect migration from host or machine identity: no such record exists, and every resume already funnels through start.
- Keep `selfreport` as a gate for anomaly notes: two switches for one Tier 2 feature could disagree, which `internal/friction` already avoids.
- Run a reflection immediately on a crash-resume note: this would spawn an agent at run entry; the next reflection covers the note anyway.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Keep the `bug` default on `lyx selfreport create`? — auto-pick: keep; manual filing is out of scope, and the reflection picks its label.
- Q2 Halt kinds as their own notes or folded into `loom-halt`? — auto-pick: folded (D3).
- Q3 Crash note timing? — auto-pick: at entry observation (D3).
- Q4 What makes a resume deliberate? — auto-pick: `lyx loom start` vouches through the handoff voucher (D4).
- Q5 Gate for anomaly notes after the knob is gone? — auto-pick: Tier 2 on (D2).
- Q6 Close #366–#382 in this task? — auto-pick: no, the hub's intake handles them.
- Q7 Edit main-weft's `loom.yaml`? — auto-pick: no; out of this role's write scope and covered by reconcile, with loadability of a stale key as an acceptance criterion.
- Q8 Does the config loader reject unknown keys? — resolved in review round 1: no. `load` fills missing keys and resolves env markers, and loom's `LoadConfig` unmarshals without `KnownFields`; D2 states the stale key keeps loading and an acceptance criterion pins it.
