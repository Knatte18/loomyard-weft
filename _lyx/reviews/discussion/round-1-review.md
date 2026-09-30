---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/discussion/decision-record.md:55
    summary: Launch-script lifecycle rests on a false premise (Down deletes reed.json, so strands do not stay persisted for Resume) and is tied to RemoveStrand alone, orphaning scripts on Down, ReplaceStrand, and failed launches
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:80
    summary: The Windows ExecutionPolicy risk contradicts the never-fail-a-launch decision; the pwsh source statement needs a decision that cannot be refused by policy, not an unguarded risk
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md:82
    summary: Q10 scrollback readers is a plan-time instruction dressed as a risk; production readers never read the launch line, so it belongs under Acceptance criteria
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md:43
    summary: The behaviour-unchanged-by-construction rationale overclaims; a sourced file skips interactive-only processing such as bash history expansion
  - id: F5
    severity: NIT
    location: _lyx/discussion/decision-record.md:83
    summary: Rejected-alternative reasoning leaks into decision-record.md, the relaunch-overwrite risk cannot occur, and two Auto-mode assumptions restate Scope Out and Decisions
---

### [BLOCKING] Launch-script lifecycle is wrong for Down and incomplete for every other strand-dropping path

**Location:** `_lyx/discussion/decision-record.md:55` (Decisions, "Removing a strand deletes its launch script"), with the matching Scope In bullet at line 17 and the Remove acceptance criterion.

**Issue:** The decision says "`Down` keeps the scripts, since the strands stay persisted for `Resume`".
That premise is false: `Down` (`internal/reedengine/lifecycle.go`) deletes `reed.json` after tearing the session down, and its own doc comment says so ("It still DELETES the state file").
A `Resume` after `Down` loads an empty state and relaunches nothing, so every script present at `Down` is orphaned permanently — the exact accumulation the decision's own rationale sets out to prevent, on the most common teardown path.
Separately, the rule is stated as "`RemoveStrand` deletes", naming one public verb rather than the lifecycle it enforces.
`ReplaceStrand` (`replace.go`) drops a strand record through `removeStrandLocked` without going through `RemoveStrand`, so a plan writer applying the decision literally leaves the replaced strand's script behind.
A launch that writes its script and then fails (split succeeded, send-keys failed) returns without persisting the strand record on the add path, orphaning a script whose GUID no state ever names.
The plan writer cannot derive the right behaviour for those paths from a rule phrased as one call site plus a false rationale.

**Fix:** State the lifecycle rule in terms of the strand record: a launch script exists only while its GUID is in reed's persisted strand table.
Derive the removal points from that rule: every strand record dropped from the table (the shared removal step behind both `RemoveStrand` and `ReplaceStrand`) deletes its script;
`Down` deletes the whole launch directory together with `reed.json`;
a launch whose strand record is never persisted deletes the script it wrote.
Correct Scope In and the Remove acceptance criterion to match, and add `Down` and `ReplaceStrand` coverage to the tests.

### [MEDIUM] Windows ExecutionPolicy risk contradicts the never-fail decision

**Location:** `_lyx/discussion/decision-record.md:80` (Open risks, first bullet), against the Write-failure decision at line 49 and the Auto-mode assumption at line 73.

**Issue:** The Write-failure decision's rationale is "a cosmetic feature must never fail a strand launch", and `CONSTRAINTS.md`'s Pane Binary Resolution clause makes the same rule binding for the prelude ("never a failed launch").
The first open risk then accepts that on a machine whose effective policy is `Restricted` or `AllSigned` the strand never starts, undetected by reed — a strand launch failed by the cosmetic feature, which the typed line never hit.
The two positions are inconsistent.
The risk is also not as remote as the Auto-mode assumption implies: `AllSigned` is a common Group Policy setting on managed machines, and `LYX_REED_SHELL` lets an operator point the pane at Windows PowerShell 5.1, whose client default is `Restricted`.
The rejected alternative dismisses the policy-proof form as "longer typed line, defeats the point on Windows", but a statement that reads and evaluates one file is still a few dozen characters plus the path, against a full line carrying several paths and flags;
it keeps the point.
The plan writer needs a decision here, since reed cannot detect the pane's policy and an unguarded risk gives it nothing to build.

**Fix:** Decide that the pwsh source statement evaluates the file's contents in the current scope in a form ExecutionPolicy does not govern — dot-sourcing a script block built from the file's text (`. ([scriptblock]::Create(<ReadFile(path)>))`), which reuses the dialect's existing `ReadFile` mechanic that already reads the prompt file.
POSIX keeps `. '<path>'`.
Drop the ExecutionPolicy risk and the RemoteSigned assumption, update the Windows acceptance criterion for `Source`, and move the old dot-source-the-file approach to the support log's Rejected alternatives with the policy reason.

### [LOW] Q10 is a plan-time instruction, not an open risk

**Location:** `_lyx/discussion/decision-record.md:82` (Open risks, second bullet); support-log Question ledger Q10.

**Issue:** "The Plan producer must check for such readers" is an instruction phrased as a risk, and Plan-Write reads only `decision-record.md`, so it needs a requirement it can verify, not an open question.
The question is also answerable at Discussion depth without enumerating call sites: shuttle's production pane readers (`requireReadyAgentPane`, `sendVerified`) classify the provider TUI's ready marker and the sent text, never the typed launch line, so the only possible readers are tests.

**Fix:** Replace the risk with an Acceptance criterion: no test or tool matches pane text against the launch line's content; any that did is retargeted to the script file or to the command's output.
Resolve Q10 in the support log's ledger with the production-reader reasoning.

### [LOW] "Behaviour unchanged by construction" overclaims

**Location:** `_lyx/discussion/decision-record.md:43` (Decisions, script content rationale), and the Goal's "Pane behaviour is unchanged".

**Issue:** Byte-equal content does not make a sourced file equivalent to a typed line.
An interactive shell processes typed input in ways it never applies to a sourced file: bash history-expands `!` on a typed line but not in a sourced file (verified this round: `echo hi!!` from a sourced file prints `hi!!`), and the history records the source statement instead of the command.
On pwsh the file route adds script-file handling the typed line never had, which is F2.
`exit` semantics, dot-source scope, and the exit status the pane shell sees afterwards match for the POSIX `.` and for pwsh's dot-sourced scope, and no launch command uses `exit` or `return`.
None of the remaining differences is a requirement beyond F2, and the POSIX live acceptance criterion tests the equivalence that matters — exports and `PATH` landing in the pane shell — so the defect is the overclaim itself, which invites a plan writer to assert a byte-for-byte equivalence the substrate does not give.

**Fix:** Scope the rationale to what holds: the pane shell executes the same statements in its own scope, so exports, `PATH`, working directory and exit status match what the typed line produced;
interactive-only processing of typed input (history expansion, history contents) no longer applies, which only removes ways the typed line could misparse.

### [NIT] Leanness: rejected alternatives and restatements in decision-record.md

**Location:** `_lyx/discussion/decision-record.md:34`, `:83`, `:75`, `:76`.

**Issue:** "needs no per-strand directory for a single file" (line 34) and "accepted rather than guarded with per-launch filenames" (line 83) are rejected-alternative reasoning; both alternatives already sit in the support log's Rejected alternatives.
The third open risk cannot occur: every relaunch path reaches `launchStrandLocked` only for a strand with no live pane (Resume and `--if-absent` select strands whose pane is not alive, and `UpdateStrand` launches a strand that has never held a pane), so no earlier pane can source a rewritten script.
Its one real requirement — the pane never reads a half-written file — is already carried by the atomic-write decision, so it passes the completeness-before-leanness test for deletion.
The Auto-mode assumptions "No operator-facing config key is added" and "`.lyx/reed/launch/` is the path; no per-strand directory" restate Scope Out and the location decision; the self-picks are already recorded in the support log's Question ledger.

**Fix:** Delete the per-strand-directory clause from the location rationale, delete the relaunch-overwrite risk, and delete the two restating Auto-mode assumptions.
Add the relaunch-overwrite reasoning to the per-launch-filenames rejected alternative in the support log, which already holds it in short form.
