---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/gatecli/cardfixer.go:135
    summary: The card fixer's new hold notifier never fires, so the code and the doc and summary claims about it describe a path that does not exist.
    fix: "In internal/gatecli/cardfixer.go remove the runner.SetNotifier call in newStrandRig, the gateProgram constant, the orchcli import and the two comment lines added above newStrandRig. In internal/gatecli/doc.go remove the two added lines that begin \"Its runner mails each held turn end of the fixer\". In _lym/landing/summary.md remove the \"Gate card fixer\" bullet and the sentence under \"The held-turn notice stays the fallback\" that names the gate card fixer, and the matching \"To check by hand\" item. The card fixer renders the no-parent directive and gatefixer.Await ends a turn that ended without a reply as dead (asking), so its question already returns to the gate's caller and needs no hold notice."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/arm.go:315
    summary: No test reaches newShed, so the wiring that turns notify_parent_on into wrapped producers, its empty-list short cut and its refusal are unpinned.
    fix: "Add one table-driven test in internal/loomcli that builds a loomCLI with cfg.NotifyParentOn set, using the package's existing loomCLI test fixture or a minimal one with recipe, env.NotifyParent, shedPaths, location and runID filled, and calls newShed. Rows: an empty list leaves every producer unwrapped; [\"<a row of the recipe>\"] makes that row's Producer a rowNoticeProducer whose paths.record equals shedrun.RowNoticeRecord(c.location, c.runID) and whose paths.status equals c.shedPaths.StatusPath; an unknown name returns a nil shed and the refusal."
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/loomcli/rownotice.go:89
    summary: A re-call of the same row after an Awaiting escalation or a resumed Stuck halt counts as a new visit and mails again, which the docs' once-per-reach wording does not say.
    fix: "Keep the code. In contracts/orchrole/orch-role.md (\"Briefing a row's agent\"), internal/loomcli/doc.go, the rowNoticeProducer and verdictCount doc comments, the shedrun.RowNotice doc comment and the notify_parent_on comment in internal/loomengine/template.yaml, state that a re-call of the row after an escalation's answer or a resumed halt counts as a new visit and mails again, since it spawns a fresh agent; a call after a yield or a pause still mails nothing."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/rownotice_test.go:101
    summary: The rownotice tests leave the wrapper's documented contracts unpinned, namely no retry after a failed post, notice before the call, pass-through of pointer and error, and warn-only on a missing status.
    fix: "Extend TestWatchRows_FailedNoticeLeavesTheCall rather than adding tests. Give countingProducer an output pointer and error field and return them, and make the refusing notify record the inner call count it sees. Assert that Call returns the inner OutputPointer and error unchanged, that notify ran before the inner call (calls was 0 when notify ran), and that a second Call with the same status posts no second notice. Add a row that removes the status file before Call and asserts that the inner call still runs and nothing is posted."
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: internal/shedrun/rownotice.go:24
    summary: RowNoticeRecord has no row in TestPathConstructors, the table that pins every shedrun path constructor, where its sibling WakeRecord has one.
    fix: "In internal/shedrun/paths_test.go add a TestPathConstructors row beside the WakeRecord row: name RowNoticeRecord, constructor func(l *lymcwd.Location) string { return RowNoticeRecord(l, \"self\") }, want filepath.Join(ScratchDir(l, \"self\"), \"row-notice.json\")."
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomengine/config.go:566
    summary: No row of TestLoadConfig_Loads pins that a notify_parent_on list decodes into Config.NotifyParentOn, as its sibling parent keys' rows do.
    fix: "In internal/loomengine/config_test.go TestLoadConfig_Loads, add an overridden row beside the code-review parent keys' rows that sets notify_parent_on to [\"Drift-Verify\"] and wants NotifyParentOn equal to []string{\"Drift-Verify\"}."
  - id: F7
    severity: LOW
    class: decision
    origin: lens:code-scope
    location: internal/loomcli/rownotice.go:28
    summary: The board entry asks to mail the parent when the run reaches a named producer or state; the diff builds rows only, the writer's own unrecorded call.
    fix: "Leave the code in place and defer this finding to the run's parent: the parent either records the rows-only narrowing, that halt states already reach it through batten and dobby notices, with lym loom decision add, or refuses it so state subscriptions are built."
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/loomcli/rownotice_test.go:96
    summary: The failure message says each notice should open with the prefix, but the check reads only the first notice.
    fix: "In TestWatchRows_TellsParentOncePerVisit, replace the lines[0] check with a loop over lines that reports each line lacking the \"[row reached] the run reached Drift-Verify \" prefix, keeping the len(lines) == 0 guard."
---

### [MEDIUM] The card fixer's hold notifier never fires

**Location:** internal/gatecli/cardfixer.go:135

**Issue:** `SetNotifier` reaches a shuttle `Run` only through `Run.Wait`.
The only caller of `handleHeldTurnEnd`, and with it of `notifyHeld`, is `internal/shuttleengine/wait.go:350`.
The card fixer never calls `Wait`.
`gatefixer.Spawn` (`internal/gatefixer/spawn.go:104`) starts the run, and `awaitFixer` (`internal/gatecli/detachfixer.go:52`) polls `gatefixer.Await`.
That function treats a turn that ended without a reply as `StatusDead` with `DeadReasonAsking` (`internal/gatefixer/await.go:62-63`).
`cardFixerSpec` (`detachfixer.go:254`) also sets no `ParentName`, so the fixer's prompt renders the no-parent directive ("When you are blocked, write your report and stop").
The added notifier is therefore dead code.
The new comment, the `gatecli/doc.go` paragraph and the summary's "Gate card fixer" bullet all describe behavior that never happens:
the summary says the fixer used to hold silently until its timeout and now mails each held turn end.
The question path itself is already closed without the notifier:
the fixer has no operator to ask, and its question ends as `asking` and returns to the gate's caller, a run agent that does have a parent.
The assistant proposed instead making the wait hold and notify. That redesigns gatefixer's one-shot cold-fixer contract, which the board entry does not ask for.
Removing the inert wiring is the smaller fix that makes the diff truthful.

### [MEDIUM] newShed's notify wiring has no test

**Location:** internal/loomcli/arm.go:315

**Issue:** The reach report lists no mapped test that reaches `loomCLI.newShed`.
The `rownotice` tests call `watchRows` directly, with paths and a poster they build themselves.
None of the following is pinned:
the activation from `c.cfg.NotifyParentOn`, the record path `shedrun.RowNoticeRecord(c.location, c.runID)`, the status paths, the `c.env.NotifyParent` poster, and the empty-list short cut.
Dropping the `watchRows` call, or passing the wrong record path, passes every test.
This is the feature's only activation point.

### [LOW] An escalation's re-call mails the row again

**Location:** internal/loomcli/rownotice.go:89

**Issue:** `verdictCount` grows by one on every appended or folded entry.
An `Awaiting` verdict appends an entry and leaves `current_producer` at the row (`internal/shedengine/run.go:470-481`), and the Bouncer returns one (`internal/shedadapters/bouncer.go:1604`).
A budget-exempt Stuck halt at a row with no `OnStuck` does the same.
The resume write appends nothing (`run.go:323-339`).
The next call of the same row therefore counts as a new visit and mails a second `[row reached]` line.
That call does spawn a fresh agent, so a second brief is useful.
But `orch-role.md`, the package doc and the template comment promise "once each time the run reaches it" and name only yield and pause as exceptions.
The docs should state the behavior the code has.

### [LOW] The wrapper's documented contracts lack test rows

**Location:** internal/loomcli/rownotice_test.go:101

**Issue:** `rowNoticeProducer`'s doc comments and the package doc promise four things the tests never check:
- the visit is recorded before the post, so a failed post is not retried;
- the notice goes out before the row's call;
- the inner outcome, pointer and error pass through unchanged;
- a missing status only warns.

`TestWatchRows_FailedNoticeLeavesTheCall` calls `Call` once, with a stub that returns an empty pointer and a nil error.
Each gap is an assertion or row in that existing test, not a new test.

### [LOW] RowNoticeRecord missing from the path table

**Location:** internal/shedrun/rownotice.go:24

**Issue:** `TestPathConstructors` (`internal/shedrun/paths_test.go:33`) states that it pins every path constructor's layout, and holds a row for the sibling `WakeRecord` (`:96-97`).
The new `RowNoticeRecord` has no row, and the reach report confirms that no test reaches it.

### [LOW] notify_parent_on decoding not pinned

**Location:** internal/loomengine/config.go:566

**Issue:** Each sibling parent key has an overridden row in `TestLoadConfig_Loads` (`internal/loomengine/config_test.go:233-240`).
`notify_parent_on` has none, so a wrong yaml tag or type on `NotifyParentOn` passes every test.

### [LOW] Rows only, where the entry names producer or state

**Location:** internal/loomcli/rownotice.go:28

**Issue:** Part 2 of the board entry asks that lym mail the parent "when the run reaches a named producer or state".
It leaves only the choice between a verb and a key to Discussion.
The diff accepts recipe rows only, and refuses anything else.
The summary's "Rows only, not states" justifies the narrowing:
halt states already reach the parent through batten and dobby notices.
But it is the writer's own call, made with no Discussion or parent decision on record.
Recording or refusing it is the parent's step, so the code stays as it is until then.

### [NIT] Prefix check reads only the first notice

**Location:** internal/loomcli/rownotice_test.go:96

**Issue:** The failure message reads "want each to open", but the check reads only `lines[0]`.
The second and third notices are never checked for their row.

## Rejected

- Assistant F2, that the unknown-row refusal breaks PATTERN-refusal-way-forward's warn-not-halt rule: a misspelt `notify_parent_on` value is invalid operator config caught at its boundary, not a guard. A silent skip would leave the parent waiting for a mail that never comes, the refusal names its way forward and has a test, and `driftengine`'s invalid-`scope` refusal is the same shape.
- Assistant F3's call for loom-run lifecycle tests through shed stepping: the history semantics it wants exercised are shedengine's, already pinned there. The real gaps, the `newShed` wiring and the wrapper contracts, are kept as F2 and F4.
- Assistant F4, that the burler role bans contradict the parent directive: the reviewer and fixer route open choices through their own files, the `decision` finding and the fixer-report's deferred section, which reach the parent through the Bouncer. The directive itself says to obey every limit the role's instructions set.
- Code-scope lens finding 2, that interactive seats still ask the operator in their pane: in those sessions the operator chose to sit in the pane, so the operator decision's premise, that the parent sees a question only when the operator relays it, does not apply. `orch-role.md` states the exception.
- Code-tests lens LOW finding on `newStrandRig`'s notifier test: moot once F1 removes the notifier.
- Mutation survivors at `rownotice.go:84` (the warning on a failed post) and `rownotice.go:92` (the `1 +` constant): the first changes only a log line, and any positive increment keeps every visit distinct, so no behavior-level test can kill the second.

## Focus departures

- Focus item 3 asks whether `SetNotifier` in `newStrandRig` sends the hold notice to the right parent with the right program name. I judged a prior question first: the notifier never fires on the card fixer's wait path (F1). The routing through `NotifyAsker` and the `GATE` program name are correct as written, but they are moot.
