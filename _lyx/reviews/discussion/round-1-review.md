---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:59-63,83-84,108
    summary: D8 finishes Done at once on unawaited payload shells while D3 reads that turn end as busy, and D5's mapping also mismatches gate waits and held-after-expiry, so the designed disagreements log as defects and no text says which side wins
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:54,59-64
    summary: D3 names three call-time facts but its rules need more, run mode and the output-file list have no named source, and the transcript interrupt marker has no time or signal to stamp a state change with
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/support-log.md:24
    summary: Rejected alternatives promises every reader the same history from the events file, but D3 folds call-time facts (process, output files, transcript) into the history, so each reader derives its own
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:129-137,171
    summary: D11 states only the false-deny side of its bound, not what the command-position regex lets through (shell wrappers, env assignments, shell keywords, other interpreters) nor that the deny is a guardrail rather than a barrier
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:113
    summary: D8 bound for a hung payload-reported shell leaves it silent up to master_timeout_min (480) because waitLive suppresses batten's quiet notice, the pane mark is not shown beside a fork, and the outcome at Timeout is unstated
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:38-43
    summary: D1 adds four hooks and an agent-writable lyx_at with no stated bound, namely that a hook prints nothing and never blocks the prompt, and that lyx_at is advisory input to a reducer that gates nothing
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:92
    summary: D6 defines live runs as those whose waiting process is alive, so a dead session whose Wait process is gone never prints, and the verb exposes api-error text without stating it
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:11
    summary: Goal part 3 says a turn end waiting on live background work is never a held stop, but D8 keeps the held stop after expiry for transcript-reported shells and the Goal does not say so
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:189
    summary: Acceptance criterion asserts existing denies' notices are byte-identical, but D11 adds a default-on python sentence to every notice, so the claim holds only with claude_deny_python false
  - id: F10
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:175-191
    summary: Decisions with observable behavior have no acceptance criterion, namely D1 first-seen fallback for a line without lyx_at, D4 reused pid and unreadable start time, D8 Master timeout bound, and D11 install for forks and interactive runs
  - id: F11
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:107-120,136
    summary: Rationale text in D8, D9 and D11 narrates history (PR-Gate finding, commit times, incident names) that carries no requirement and belongs in support-log.md
  - id: F12
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:56
    summary: Q14 says the llm-tier scenario captures Notification, while the acceptance criterion lists UserPromptSubmit, Notification and SessionEnd, and neither says which Notification types a bypass-mode run can reach
---

### [MEDIUM] D8's immediate Done and other designed mismatches log as defects under D5's mapping

**Location:** `decision-record.md` D3 (lines 59-63), D5 (lines 83-84), D8 (line 108).
**Issue:**
D8 finishes a run Done at once when every output file exists and only unawaited payload-reported shells are outstanding.
D3 defines `idle-done` as a turn end with no outstanding work, and a turn end with outstanding work as `busy` (cause `background`), so the same turn end reads `busy`.
D5's mapping pairs `done` with `idle-done` and logs `shuttle: session state disagrees` at Warn, so every such finish logs a disagreement that D8 itself designs.
Two more mismatches follow from the code I read in `wait.go` and `waitmark.go`.
A gated run's wait (label `gate <entry>`) happens after a turn end with every output file present, which D3 reads `idle-done`, while the mapping pairs `waiting` with `busy`.
A held turn end after a transcript-reported shell expires is `held` in the wait loop, but D3 still reads `busy` (outstanding work), and the mapping pairs `held` with `idle-stalled` or `asking`.
Neither D5 nor the acceptance criteria say which side wins or that these are expected.
The acceptance criterion "logs a disagreement once when its own classification and the state differ under D5's mapping" makes the Warn mandatory for all three.
The focus question is settled this way: as written it reads as a contradiction, not as a designed disagreement.
**Fix:**
Decide, in D3 or D5, which side is authoritative where the wait loop deliberately acts on a turn end the model calls `busy` or `idle-done`.
Either exempt the three cases from the Warn (and say so in the criterion), or amend D3 so that `idle-done` covers an output-complete turn end whose only outstanding work is unawaited payload shells, and give the gate wait and the post-expiry hold their own rows in D5's mapping.

### [MEDIUM] D3's call-time inputs are miscounted and partly unsourced

**Location:** `decision-record.md` D3 (line 54, lines 59-64).
**Issue:**
D3 says the reducer folds signals "plus three facts read at call time".
The rules need these inputs beyond the signals: provider-process liveness (D4), the output-file list, the run mode (autonomous or interactive), a transcript interrupt marker, and a transcript API-error marker.
That is five, and the decision names no count-matching list.
Run mode and the output-file list have no named source.
The run record already holds them (`Interactive` and the output files in `rundir.go`'s record), but the Decision does not say the reducer reads the run record, and `lyx shuttle state` and the orch watcher read through other paths.
The interrupt marker has no signal and so no `lyx_at`, yet D3 stamps a change "with the signal's `lyx_at`, or, for `dead` and `unknown` from a call-time fact, with the time of the reading".
The `idle-stalled` causes `interrupt`, `no-output` and (transcript-derived) `api-error` fit neither stamp, so their `since` is undefined.
A "pure reducer" that reads a registry, a transcript and the output files is also not pure; the decision should name which parts are pure.
**Fix:**
Replace "three facts" with the named list, give each a source (the run record for mode and output files, `claudeengine` for the registry and the transcript markers), and state how a state caused by a transcript marker or a missing output file is stamped.

### [MEDIUM] "Every reader gets the same history" does not hold under D3

**Location:** `support-log.md` Rejected alternatives (line 24) against `decision-record.md` D3 (lines 54, 67).
**Issue:**
The rejection of a persisted `state.yaml` rests on "deriving from the timestamped events file gives every reader the same history with no new writer".
D3 derives `dead` and `unknown` from facts read at call time and stamps them with the time of the reading.
Output-file presence is also read at call time, so the same turn end reads `idle-done` or `idle-stalled` depending on when it is read.
Two readers (the wait loop, `lyx shuttle state`, batten) therefore produce different histories and different `since` values for the same run, and a state change that exists only in the process that observed it is lost when that process ends.
**Fix:**
Correct the rationale in the support log to say the history is reproducible only for the signal-derived states, and state in D3 that call-time entries are per-reader.
If a stable history is a goal, say which writer records the call-time entries; this is a decision for the author.

### [MEDIUM] D11 does not state what the python deny lets through

**Location:** `decision-record.md` D11 (lines 129-137) and Open risks (line 171).
**Issue:**
The attack-surface question asks what a new guard lets through and what bounds it.
D11's Bound covers only the false-deny side (a legitimate Python run, with `claude_deny_python: false` as the switch).
The Open risks line adds `py` and `uv run` as misses, but the Decision, which is what the plan writer reads, names neither.
The command-position list also leaves out cases an agent reaches in ordinary use: `bash -c "python3 …"` and `sh -c`, `env FOO=1 python3` (assignments between `env` and the command), shell keywords (`then`, `do`, `{`, `!`), `nohup`, `command`, `timeout`, and `python -m …` through `pipx` or `uvx`.
The regex runs over the JSON payload, so a heredoc line that begins `;python3` or a quoted string with `&& python3` inside it matches as command position.
The Rationale states the operator's rule as "lyx agents never use Python", while the mechanism is a best-effort guardrail; the Decision should say so, since the plan writer and the test author read the rule as total.
The Decision also does not say who can flip the per-repo switch or when it is read; the key sits in config an agent could edit, and the read time (spawn) is what bounds that.
**Fix:**
State in D11 that the deny is a guardrail with named let-through classes (wrappers, interpreter launchers, other scripting runtimes), name the false-deny class (quoted or heredoc text), say the switch is read at spawn, and add the wrapper rows the criterion should pin as accepted misses or as covered.

### [MEDIUM] D8's bound leaves a hung shell invisible and its Timeout outcome unstated

**Location:** `decision-record.md` D8 (line 113), D7 (line 100).
**Issue:**
Settled for the focus question: the Bound does cover the Master case (`master_timeout_min`, 480 in `websterengine/template.yaml`) and the turn end stays free of any parent notice, since D8 says it is never notified.
What the Bound does not cover: the 10-minute expiry was also the only prompt for a parent notice about a stuck payload-reported shell, and D8 removes it.
D7 keeps `waitLive` counting background-shell waits, so batten's quiet notice is also suppressed while the shell hangs.
That leaves a pane mark as the only signal, for up to eight hours for a Master.
`shellWaitActive` in `waitmark.go` returns false whenever the outstanding list holds a fork, so with a fork and a hung shell together no mark shows, contrary to "the mark shows it the whole time".
The Bound names `verify-watchdog` as the owner of a step that never returns, but that task is listed Out of scope and is not built.
The Decision also does not state what happens at Timeout (the outcome, and whether the shell is reaped or left).
**Fix:**
State the mixed fork-and-shell case, state the Timeout outcome, and either bound visibility (for example a log line when a payload-reported shell passes `background_shell_wait_min`, which changes no decision) or record in the Decision that a hung shell is silent until Timeout by choice.

### [LOW] D1 states no bound for its new hooks and the agent-writable stamp

**Location:** `decision-record.md` D1 (lines 38-43).
**Issue:**
Four new hooks run on every prompt, notification, API error and session end.
A `UserPromptSubmit` hook's standard output is added to the model's context and its exit status can block the prompt, so the Decision should state that the hook command prints nothing and a failed append never blocks the turn.
The existing Stop command ends `cat >> file && printf '\n' >> file`, which meets that today; the new commands are not said to.
`lyx_at` is written by a shell command and the events file sits in a run directory the agent can write in bypass mode.
The record never says the stamp is advisory, that no decision reads it (D5's Bound implies it), or what a missing or malformed stamp does beyond "first seen".
**Fix:**
Add a Bound line to D1: the hook command prints no output and is non-blocking; `lyx_at` is advisory, read only by the reducer, and a malformed value reads as first-seen.

### [LOW] D6 leaves the dead state unreachable and does not state its exposure

**Location:** `decision-record.md` D6 (line 92).
**Issue:**
"Live as `ReadAgentActivity` defines it" means the record reads running and the waiting process is alive.
A session whose Claude process died is `dead` only while Wait runs; once Wait is gone the run is not live and the verb does not print it.
The verb is stated as showing state "without grepping logs", so the author should decide whether `dead` and crashed runs are in scope for it.
The printed cause for `idle-stalled` `api-error` carries the error text, which the decision does not list among the verb's outputs.
**Fix:**
Decide whether the verb also lists non-live running records, and list the error text among the fields printed.

### [LOW] Goal part 3 overstates D8

**Location:** `decision-record.md` Goal (line 11), D8 (lines 106-109).
**Issue:**
Goal part 3 says a turn end waiting on live background work is "never a held stop or a question".
D8 keeps the held stop for transcript-reported shells after `background_shell_wait_min`, and Done-at-once for output-complete runs.
**Fix:**
Qualify part 3 to payload-reported work.

### [LOW] "Byte-identical" for the existing denies does not follow from D10 and D11

**Location:** `decision-record.md` Acceptance criteria (line 189).
**Issue:**
D10 says the existing denies move "unchanged in behavior".
The criterion asserts the hooks and notices are byte-identical.
D11 installs the python deny by default and announces it in the notice, so the notice for every default run gains a sentence, and a run that previously had no notice now has one.
The criterion is true only for the hook commands, or with `claude_deny_python: false`.
**Fix:**
Restate the criterion as: each existing deny's hook command and notice sentence is unchanged, and the notice differs only by the python sentence.

### [LOW] Decisions without an acceptance criterion

**Location:** `decision-record.md` Acceptance criteria (lines 175-191).
**Issue:**
No criterion pins: a hook line without `lyx_at` reading at first-seen time (D1); a reused pid or unreadable start time reading `unknown` (D4, only "no registry entry" is covered); the Master timeout bound of D8; and the python deny being installed for forks and for interactive runs (D11).
The ordering rule "output files before any negative state" (D3) is also unpinned, apart from the idle-done case.
**Fix:**
Add each as a row or assertion in the criterion that covers the neighbouring behavior, not as new criteria.

### [LOW] Rationale narrative in D8, D9 and D11 belongs in the support log

**Location:** `decision-record.md` D8 (line 112), D9 (line 120), D11 (line 136).
**Issue:**
Relocation, after the completeness-before-leanness test.
D9's Rationale compares commit times (batchifier 15:44, `shuttle-turns` 18:23 on 2026-10-07; I confirmed both in `git log`).
The requirement it supports, the regression pin and the statement that the old path is gone, is already in D9's Decision text, so only the timeline moves.
D8's "(the `shuttle-turns` PR-Gate finding)" and D11's "after the `board-model` 75-minute stdin hang and the `agents-no-scripted-edits` script edit" are incident history.
Their requirements, Done at once with outputs present and the python deny itself, are stated in the Decisions.
Support-log Interview items 8 and 10 already carry the substance.
**Fix:**
Cut the three narratives from `decision-record.md` and keep one clause per Rationale for the premise (the payload is authoritative; the old branch is gone), adding the commit times to Interview item 8.

### [NIT] Q14 and the llm-tier criterion name different payload sets

**Location:** `support-log.md` Question ledger Q14 (line 56), `decision-record.md` Acceptance criteria (line 191).
**Issue:**
Q14 says the scenario captures `Notification`; the criterion says `UserPromptSubmit`, `Notification` and `SessionEnd`, and Q14 and the Auto-mode assumption agree that `StopFailure` stays schema-derived.
Lyx agents run in bypass mode, so a `permission_prompt` notification cannot be captured there; only `idle_prompt` is reachable, and neither file says so.
**Fix:**
Make Q14 name all three payloads, and say which `Notification` types the scenario can capture.

Checked and not findings: the Q1-Q13 ledger answers match their Decisions; `StopFailure` staying schema-derived is consistent between Q14, the Auto-mode assumptions, Open risks and the criterion; the removed `shedadapters` branch is gone from the tree (`master is asking` has no hit in the Go or Markdown files); and none of the findings asks for rejected alternatives, signatures or call-site lists, per the rubric's `Do not flag` list.
