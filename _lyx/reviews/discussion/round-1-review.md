---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:97
    summary: The AskUserQuestion notice defers to a deny reason the session has not seen at spawn, so it gives no actionable steer before the first refusal.
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:107
    summary: D7 justifies its constraint by the JSON-echo check rather than the notice's own path, and omits the single-line requirement that path imposes.
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:96
    summary: The fork-run notice reaches the forks too, through the inherited system prompt, and there it invites a fork to spawn further forks.
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md:105
    summary: The rationale for not announcing the webster guard assumes only the Master receives the notice and that the guard is Master-only; neither holds.
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md:104
    summary: The resume-line rationale contradicts the record's own snapshot claim, and Open risks bullet 1 is rationale rather than a risk.
  - id: F6
    severity: LOW
    location: _lyx/discussion/decision-record.md:72
    summary: D4 lets a preparation amend the prompt without bound while D5 appends; the amendment contract is not pinned to append-only.
  - id: F7
    severity: LOW
    location: _lyx/discussion/decision-record.md:157
    summary: The stale loomengine plan.go file comment is a required doc fix but sits only in the optional notes, absent from Scope and Acceptance criteria.
  - id: F8
    severity: LOW
    location: _lyx/discussion/decision-record.md:129
    summary: The last Auto-mode assumption duplicates Open risks bullet 3 and contradicts it by calling the outcome correct in every case.
  - id: F9
    severity: LOW
    location: _lyx/discussion/decision-record.md:135
    summary: Open risks bullet 2 is a mitigated scenario, not a risk; the requirement it implies (00-overview.md is never copied from the archive) belongs in D2.
  - id: F10
    severity: NIT
    location: _lyx/discussion/decision-record.md:102
    summary: D6 enumerates the spawned agents, a perishable and already incomplete consumer list.
---

### [MEDIUM] AskUserQuestion notice points at a steer the session has not seen

**Location:** `_lyx/discussion/decision-record.md:97` (D6, AskUserQuestion bullet)

**Issue:** The notice says "the steer already given by the deny reason applies".
The deny reason (`steerAskUserQuestionDeny` in `internal/shuttleengine/claudeengine/settings.go`) reaches the session only as the `permissionDecisionReason` of a refused call.
At spawn, before any refusal, the session has seen no such steer, so the sentence tells it nothing it can act on.
That is the same friction the task fixes for the Agent tool: the session learns the channel only by losing a call.

**Fix:** Make the D6 sentence carry the steer's content: `AskUserQuestion` is unavailable;
when blocked or needing operator input, state the question as the final message and end the turn without writing the result file.

### [MEDIUM] D7's rationale does not match the notice's path, and the path requires a single line

**Location:** `_lyx/discussion/decision-record.md:107-111` (D7), and the acceptance criterion at line 149

**Issue:** The existing forbidden-character check guards steer text inside `echo '<json>'`, where a single quote ends the shell literal and a double quote or backslash corrupts the JSON.
The notice travels a different path: `buildLaunchCmd`/`buildResumeCmd` pass it through `shell.Shell.Quote` as a command-line argument, and reed types the whole line into the pane with `tmux send-keys -l` followed by `Enter` (`internal/reedengine/spawn.go`).
On that path:

- A single quote is harmless: both `posixShell.Quote` and `pwshShell.Quote` escape it.
- Backticks and `$` are literal inside single quotes on both shells, so D6's backtick-wrapped tool names are safe under pwsh.
- A double quote or backslash can be mangled by Windows native-argument passing from pwsh, which is a real reason to keep them out.
- A newline or carriage return submits a partial command the moment send-keys types it, and D7 does not forbid it.
  D6's "one sentence per installed deny" invites joining sentences with newlines.

The inherited set `'"\` is conservative and fine to keep, but the rationale is wrong and the one character class this path breaks on is missing.

**Fix:** Restate D7's decision: the notice is a single line (sentences joined by a space), free of line breaks, single quotes, double quotes and backslashes.
Restate the rationale by the actual path: send-keys submits on a line break, and Windows native-argument passing mangles double quotes and backslashes;
the single quote is kept out for parity with the steer check.
Extend the acceptance criterion to cover line breaks in the assembled notice.

### [MEDIUM] The fork-run notice is inaccurate for the forks that inherit it

**Location:** `_lyx/discussion/decision-record.md:96` (D6, fork-run bullet)

**Issue:** An appended system prompt is part of the session's system prompt, which fork subagents inherit.
Fork-mode runs are the webster Master and every burler cluster round (`burlerengine`'s `ForkSubagents: p.ClusterFan != ""`), so the notice reaches every fork they spawn.
"Only fork subagents may be spawned" tells a fork it may spawn forks, while a fork is meant to do its own batch or lens work (`webster-prefix-fork.md`).
A fork acting on it either re-delegates or loses a call to a refusal — the friction this task exists to remove.

**Fix:** Word the fork-run sentence so it holds in every session that receives it:
the `Agent` tool accepts only fork subagents and refuses every other subagent type;
a fork does its own work and never spawns further subagents.
Add an acceptance criterion pinning that the fork-run notice carries the no-re-delegation clause.

### [LOW] The webster-guard exclusion rests on a false premise

**Location:** `_lyx/discussion/decision-record.md:105` (D6 rationale, last sentence), and Scope Out at line 33

**Issue:** The rationale says announcing the guard to the Master would mislead it.
The guard is installed for every fork-mode run, burler cluster rounds included, and a system-prompt notice reaches the forks as well as the Master, so a sentence scoped to forks ("inside a fork, `lyx webster` is refused") would mislead no one.
The exclusion itself still holds for a different reason: webster's forks are already told never to run `lyx webster` by their own stencils (`webster-prefix-fork.md`, `webster-template-integration.md`), and burler forks have no reason to.

**Fix:** Keep the exclusion; replace the rationale with the real one: webster's fork stencils already carry the rule, and the guard is a backstop for a case the prompts cover.

### [LOW] Resume-line rationale contradicts the snapshot behaviour; Open risks bullet 1 is not a risk

**Location:** `_lyx/discussion/decision-record.md:104` (D6 rationale), and Open risks bullet 1 at lines 133-134

**Issue:** D6 says the resume line carries the notice "for the same reason it already carries model, effort and permission mode".
Model and effort are not persisted across a resume, which is why `buildResumeCmd` carries them;
the appended system prompt is, by the record's own Auto-mode assumption and `claude --help` (`--system-prompt-snapshot`, default on).
The real reasons are that the snapshot is re-rendered from the current flags after a compaction, and that recording is not enabled everywhere;
without the flag on the resume line, a resumed session loses the notice at its next compaction.
Open risks bullet 1 then calls its own scenario harmless, so it is rationale, not a risk.

**Fix:** Rewrite the resume sentence of D6's rationale around the compaction re-render and the recording-not-enabled case.
Delete Open risks bullet 1; its one useful fact (a notice change never reaches a running session, which matches the hooks) is covered by the same rewritten sentence.

### [LOW] The amendment contract is not pinned to append-only

**Location:** `_lyx/discussion/decision-record.md:72-75` (D4), against D5 at line 84

**Issue:** D4 widens the preparation so it "can amend the prompt", which admits an arbitrary rewrite;
D5's only consumer appends.
The byte-identical criteria are testable either way, but under an unbounded rewrite "makes no amendment" has no crisp meaning, and a future preparation could clobber the composed prompt the attach-unchanged reasoning relies on.

**Fix:** Pin D4 to append-only: a preparation contributes text appended to the end of the composed prompt, and contributing nothing leaves the prompt byte-identical.

### [LOW] The stale plan.go comment is a required fix hidden in the optional notes

**Location:** `_lyx/discussion/decision-record.md:157` (Notes for the plan writer, third bullet)

**Issue:** `internal/loomengine/plan.go`'s file comment says rotating a stale `_lyx/plan/` is "the future loom phase machine's responsibility", which is false now that `NewPlanDirRotator` owns it, and this task changes what that rotation does.
The notes section is optional and non-binding, and Scope's docs line names only "the package docs of the touched packages", which need not include `loomengine`.
Under the completeness-before-leanness test, a known-false doc sentence the task should fix belongs in Scope and Acceptance criteria.

**Fix:** Add the `plan.go` file comment to Scope's docs bullet and to the docs acceptance criterion;
drop the third notes bullet.

### [LOW] Auto-mode assumption duplicates and contradicts Open risks bullet 3

**Location:** `_lyx/discussion/decision-record.md:129`, against line 136

**Issue:** Both describe a restarted run whose plan directory holds a plan the new session is told to reuse.
The assumption claims the decision-record check "makes this correct in every case";
the risk says the same check "relies on the agent following it".
The risk is the accurate statement.

**Fix:** Delete the Auto-mode assumption bullet and keep Open risks bullet 3.

### [LOW] Open risks bullet 2 is a mitigated scenario hiding a D2 requirement

**Location:** `_lyx/discussion/decision-record.md:135`

**Issue:** The bullet concludes the scenario is caught, so it is rationale, not an open risk.
It also implies an unstated requirement: D2 says to carry cards forward by copying, but a session that copies the whole archive copies `00-overview.md` with it, including its `approved` flag, before the cards are revised.
Plan-Write always writes `approved: false`, and an archive from a restarted run may hold a plan Plan-Bouncer already approved.

**Fix:** Extend D2: `00-overview.md` is never copied from the archive;
it is written fresh, last, with `approved: false`.
Delete Open risks bullet 2.

### [NIT] Perishable consumer list in D6

**Location:** `_lyx/discussion/decision-record.md:102`

**Issue:** "(Discussion, Plan, Describe, Burler, Bouncer judges, webster Master)" enumerates the current spawners, and already misses others that run through the same engine (the friction reflection spec, the loom driver spec).

**Fix:** Say "every session `claudeengine` spawns", without the list.
