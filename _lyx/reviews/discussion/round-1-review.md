---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Constraints, Config Strictness bullet; Acceptance criteria)
    summary: the strand_name retirement cites a reed retired-key mechanism that does not exist for reed.yaml, and no acceptance criterion pins what a stale strand_name key does
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Open risks, custom-title bullet; Acceptance criteria, Claude session-name provider bullet)
    summary: the open risk says a transcript-format change degrades drift detection to never detecting, but the provider criterion reports drift when no custom-title entry exists, so the real degradation is a /rename typed on every idle tick, and an unreadable transcript has no criterion
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (Parentage, hand-started hub bullet)
    summary: the ly hub workflow documents it names a deliverable no Scope item covers and no repo file holds, so the plan writer cannot tell whether to write it or where
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md (Parentage; Auto-mode assumptions; Acceptance criteria, reed bullet)
    summary: the worktree parent is read from the self seed, which is right only through shedrun's alias; the default run's directory is _lyx/shed/<slug>/ and the record never says so
  - id: F5
    severity: LOW
    location: _lyx/discussion/support-log.md (Interview turn 15; Rejected alternatives, --parent bullet; Question ledger Q13)
    summary: turn 15, Q13 and the --parent rejection still give the pre-hub-review position that a hand-started session cannot set env and lyx orch start is the only hub path
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md (Acceptance criteria)
    summary: the spawn refusal for a task worktree whose name fails the slug grammar and the reed and fabric package docs named in Scope have no acceptance criterion
  - id: F7
    severity: NIT
    location: _lyx/discussion/decision-record.md (Acceptance criteria, last bullet)
    summary: the hand-started hub seed criterion restates the Seed criterion, since a per-command env prefix and an inherited env are the same input to batten run
  - id: F8
    severity: NIT
    location: _lyx/discussion/decision-record.md (Parentage, hand-started hub rationale; Delivery rationale)
    summary: the hub-review attribution and the 2026-09-30 date are deliberation history that carries no constraint for the plan writer
---

### [MEDIUM] `strand_name` retires through a mechanism that does not exist

**Location:** `decision-record.md`, Constraints (Config Strictness Invariant bullet) and Acceptance criteria.

**Issue:** The record says `strand_name` "retires through reed's existing retired-key mechanism".
`internal/reedengine/config.go` unmarshals `reed.yaml` with plain `yaml.Unmarshal`, so an unknown key is ignored on load;
`lyx config` reports such a key as preserved, and `lyx config reconcile --apply` removes it.
The only retired-key precedent in reed, `retiredkeys_test.go`, covers display keys in `reed.json` state, not `reed.yaml`.
The Config Strictness Invariant itself governs `Load` versus `LoadOrTemplate`, not key retirement.
A plan writer hunting for the named mechanism finds nothing, and no criterion pins the observable behaviour of an operator's `reed.yaml` that still carries `strand_name`.

**Fix:** State the concrete behaviour: the key leaves `Config` and both templates, a `reed.yaml` still carrying it keeps loading with the key ignored (pinned by a test, as reed's retired display keys are), and `lyx config reconcile` removes it.
Add that as an acceptance criterion.

### [MEDIUM] The session-name provider's degradation is stated backwards

**Location:** `decision-record.md`, Open risks (`custom-title` bullet) and Acceptance criteria (Claude session-name provider bullet).

**Issue:** The criterion says a transcript with no `custom-title` entry reports drift.
The open risk says that if the entry's format changes, "drift detection degrades to never detecting".
Under the criterion's rule, a format change makes every read find no entry, so every idle tick reports drift and the watchdog types `/rename` again, forever.
The open risk also asks the repair to "log that it could not read the name rather than fail", but no criterion covers a missing or unreadable transcript, and the record does not separate that case from "no entry yet".

**Fix:** Split the two cases in the criterion: no `custom-title` entry reports drift; a missing or unreadable transcript logs that the name could not be read and reports no drift.
Correct the open risk to the real degradation: a changed entry format reads as no entry, so the watchdog retypes `/rename` on each idle tick and logs each repair.

### [MEDIUM] "The ly hub workflow documents it" has no home

**Location:** `decision-record.md`, Parentage, the hand-started hub bullet.

**Issue:** The bullet ends "and the ly hub workflow documents it".
Scope's In list names `CONSTRAINTS.md`, `docs/overview.md` and the reed and fabric package docs; no ly hub workflow.
The repo holds no such document: the `ly` plugin carries only `ly-drive`, and `docs/` has no hub-session guide.
Scope's Out list also excludes names for hand-started Claude sessions.
The plan writer cannot tell whether this is a deliverable, and if so where it goes.

**Fix:** The env-prefix path is the hub session's own practice until `orch-live-ready`, and the bullet already says it needs no code.
Say it is not a deliverable of this task: the hub session applies it from this record.

### [LOW] The worktree parent's seed is named only by the `self` alias

**Location:** `decision-record.md`, Parentage ("the `parent` of its default run (`self`)"), Auto-mode assumptions ("its `self` run's seed"), Acceptance criteria (reed bullet, "the worktree's `self` seed").

**Issue:** Focus 1 asked whether this names the wrong seed.
It does not: `shedrun.SelfRunID` is still the default run-id, and `ResolveRunID` maps it to the worktree's slug, so the seed lives at `_lyx/shed/<slug>/seed.json` (this worktree's own run sits at `_lyx/shed/agent-naming/`), with `_lyx/shed/self/` only a read-only legacy fallback.
Reading through `shedrun` with `self` reaches the right file.
But the record never says so, and the writer stencil itself calls `_lyx/shed/self/` legacy; a plan writer who takes "the `self` seed" literally reads a directory that new runs never create.

**Fix:** Name the default run as run-id `self`, resolved by `shedrun` to the worktree's slug, read through `shedrun` rather than a joined `self` path.

### [LOW] The support log still carries the superseded hub position

**Location:** `support-log.md`, Interview turn 15, Rejected alternatives (`--parent` bullet), Question ledger Q13.

**Issue:** Turn 15 says a hand-started session cannot export env to its Bash calls, and picks `lyx orch start` alone; Q13 resolves to `lyx orch start` alone.
The hub review then added the env-prefix path, and the decision record now carries it.
The `--parent` rejection's reason, "`lyx orch start` is the supported hub", no longer is the whole reason: the env prefix gives a hand-started hub a parent without a flag.
The Review rounds entry records the change, but the ledger is meant to hold each question's resolution, and Q13 now disagrees with the record.

**Fix:** Keep turn 15 as the interview happened, with a pointer to the hub review.
Update Q13 to the current resolution and the `--parent` rejection's reason to the env-prefix path.

### [LOW] Two observable decisions lack a criterion

**Location:** `decision-record.md`, Acceptance criteria.

**Issue:** Name grammar decides that a task worktree whose name fails the slug grammar refuses a spawn with a way forward; the `agentname` unit criterion covers rejecting a bad slug, but nothing covers the spawn refusal and its remedy.
Scope names the reed and fabric package docs, but the docs criterion names only `CONSTRAINTS.md` and `docs/overview.md`.

**Fix:** Fold the bad-slug spawn refusal into the existing uncoded-hub refusal criterion, and add the reed and fabric package docs to the docs criterion.

### [NIT] The hand-started hub seed criterion restates the Seed criterion

**Location:** `decision-record.md`, Acceptance criteria, last bullet.

**Issue:** A per-command prefix `LYX_STRAND_NAME=<code>:orch lyx batten run <slug>` and an inherited `LYX_STRAND_NAME` are the same input to `batten run`, which cannot tell them apart.
The earlier Seed criterion already pins that `batten run` under `LYX_STRAND_NAME` records it as parent.
"From a shell with no other naming" adds no condition the earlier test lacks.

**Fix:** Remove the bullet.

### [NIT] Dated deliberation history in two rationales

**Location:** `decision-record.md`, Parentage (hand-started hub rationale, "(hub review, 2026-10-02)") and Delivery rationale ("on 2026-09-30").

**Issue:** The completeness test first: the Parentage rationale's substance (env set by reed is the only source lyx reads; without the path no run has a parent and `parent-review` skips every review) is a constraint the plan writer needs and stays.
The attribution "(hub review, 2026-10-02)" carries nothing for the plan writer; the support log's Review rounds already records it.
The Delivery rationale's evidence, that SendMessage already addresses sessions by name, stays; the date is history.

**Fix:** Drop the attribution and the date, keeping both rationales' substance.

### Focus items judged not a defect

- **`-N` numbering and a freed number.**
  The rule ("the lowest free `-N` … counted against every strand in that worktree's state, live or dormant") already settles it: a number frees when its strand leaves reed state, and the next spawn of that role takes it.
  Reuse cannot misroute a message meant for the earlier holder in practice, since that holder's session no longer exists, and only `orch` and `driver` are addressed by convention, both first instances under bare names.
  For those, reuse is the wanted behaviour: a relaunched hub keeps the address its children's `LYX_PARENT` holds.
- **Scope's manifest-retire explanation.**
  It carries a constraint the plan writer needs: CLAUDE.md's task-completion rule otherwise asks for the module doc under `manifest/designs/` to be updated, and this line, with its reason, is what licenses leaving `reed-mailbox.md` alone and backs the criterion that nothing under `manifest/` changes.
  It stays.
