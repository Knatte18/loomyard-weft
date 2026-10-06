---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/run.go:456
    summary: Comments the diff adds or changes in shuttleengine (run.go, doc.go, engine.go, spec.go, fakes_test.go, startup_test.go) keep a comma before and, so or which with its own subject on one line
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/shuttleengine/claudeengine/skillload.go:1
    summary: New comments in skillload.go and the changed paragraph in claudeengine/doc.go keep independent clauses joined by a comma and and or so on one line
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/orchengine/doc.go:180
    summary: Changed comments in orchengine/doc.go, orchengine/watcher_test.go and the two loomcli stub-script tests keep independent clauses joined by a comma and and or so on one line
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/orchengine/watcher_test.go:921
    summary: No test reaches the retry step's timeout, which skips only the retry list with cause timeout and types no second retry
  - id: F5
    severity: NIT
    class: consistency
    location: internal/orchengine/watcher_test.go:904
    summary: The reset assertion compares ReloadStep with a literal 0 where the diff introduced ReloadStepSkills
---

Range: no live rework round exists (`_lyx/loom/rework/` is absent), so the range is `git diff $(git merge-base main HEAD)..HEAD`, with `product.parent` `main` read from `_lyx/shed/skills-in-one-turn/status.json`.

Mechanical state: `go build ./...` passes, `go vet -tags llm` over the plan's packages is clean, and `go test` passes untagged, with `-tags integration` and with `-tags tmux` over the five packages the plan's verify names.
Card 4's verify (`grep -rn 'SkillUnknown\|SkillLoadSequence' --include=*.go internal`) is empty, and no doc, contract or stencil outside the touched packages still describes the per-skill `/<skill>` load.

Focus answers:

- Comment breaks: not met; F1, F2 and F3 list the places.
- Parallelism of the two new reload tests: met.
  `TestWatcher_ReloadRestartAroundTheRetryStep` and `TestWatcher_ReloadReadsAnUnreadableStepAsThePointer` (and its rows) call `t.Parallel`; `TestWatcher_SkillSkipCauses` and `TestStartup_SkillLoadTurns` name the process-global logger as their reason for not doing so.
- Empty `reload_retry` branch: met.
  The `retry step with an empty retry list` row of `TestWatcher_ReloadReadsAnUnreadableStepAsThePointer` fails on an implementation that types `LoadSkills` with an empty list, because the second call would be `skills:` instead of the pointer `send:`.
- Orch warn vocabulary: met for each cause the table drives.
  `TestWatcher_SkillSkipCauses` asserts the skipped count and the log fragments for `unknown`, `not loaded`, `timeout` and `skill load unverified`, so an implementation that logs nothing fails; the retry step's timeout is the one cause path with no row (F4).
- `ReloadStep` constants: met in production code.
  `state.go` and `watcher.go` assign and compare only named constants; the one leftover literal is a test assertion (F5).

### [BLOCKING] Shuttleengine comments keep independent clauses on one line

**Location:** `internal/shuttleengine/run.go:456`, `:458`, `:522`; `internal/shuttleengine/doc.go:94`, `:95`, `:96`, `:98`; `internal/shuttleengine/engine.go:182`; `internal/shuttleengine/spec.go:140`; `internal/shuttleengine/fakes_test.go:397`, `:427`, `:428`; `internal/shuttleengine/startup_test.go:1082`
**Issue:** `PATTERN-comment-line-breaks` requires a break at every independent-clause boundary in a new or changed comment.
Each listed line is new or changed by the diff and holds a comma before `and` or `so`, with its own subject and verb after it:
`run.go:456` (`..., and an unreadable turn is confirmed unverified`), `:458` (`..., so Wait never reads one`), `:522` (`ended is false on a timeout, and died is true when ...`);
`doc.go:94` (`..., and the missing ones are asked for once more ...`), `:95` (`..., and a turn that never ends skips its skills ...`), `:96` (`..., and a skipped skill never fails or hangs it`), `:98` (`..., so Wait never reads one`);
`engine.go:182` (`..., and the orch watcher's per-tick reload uses it ...`);
`spec.go:140` (`..., and a non-empty list on an engine without one is refused at start`);
`fakes_test.go:397` (`..., and ClassifySkillLoad answers ...`), `:427` (`..., so a verified send finds its text, and which plays ...`), `:428` (`..., and any other appends a turn end ...`);
`startup_test.go:1082` (`..., so Wait never classifies one`).
**Fix:** Break each line before the conjunction so the second clause starts its own line, keeping the wording.

### [BLOCKING] Claudeengine comments keep independent clauses on one line

**Location:** `internal/shuttleengine/claudeengine/skillload.go:1`, `:24`, `:113`; `internal/shuttleengine/claudeengine/doc.go:58`
**Issue:** The same rule is broken in the new file's comments and the changed doc paragraph:
`skillload.go:1` (`..., and the transcript a Stop payload names says which loads happened`), `:24` (`..., and only main-chain entries after it count`), `:113` (`..., so the caller widens the window`);
`doc.go:58` (`..., and a skill is loaded when its Skill call is answered ...`).
**Fix:** Break each line before the conjunction so the second clause starts its own line.

### [BLOCKING] Orchengine and loomcli comments keep independent clauses on one line

**Location:** `internal/orchengine/doc.go:180`, `:189`, `:194`, `:195`; `internal/orchengine/watcher_test.go:917`; `internal/loomcli/integration_driverbootstrap_test.go:51`; `internal/loomcli/smoke_driverstrand_test.go:71`
**Issue:** The same rule is broken in the changed reload-sequence section and in the stub-script comments:
`doc.go:180` (`..., so the resuming phase restores them: ..., and the one-line pointer follows`), `:189` (`..., and the pointer follows`), `:194` (`..., and there is no second retry`), `:195` (`..., so a restart before the move reads ... and a restart after it never retries twice`);
`watcher_test.go:917` (the field comment `the skills turn never ends, so its timeout passes`);
`integration_driverbootstrap_test.go:51` (`..., so shuttle confirms the load unverified at once`);
`smoke_driverstrand_test.go:71` (`The driver spec names skills, and shuttle types one skill-load message ...`).
**Fix:** Break each line before the conjunction, and rephrase the field comment so it holds one clause, for example as a compound predicate.

### [MEDIUM] The retry step's timeout is uncovered

**Location:** `internal/orchengine/watcher_test.go:921`
**Issue:** Card 3 and `orchengine/doc.go` say a retry step that passes the handoff timeout skips its skills with cause `timeout` and types no second retry.
The `timeout` row of `TestWatcher_SkillSkipCauses` reaches only the skills step, whose skip list is `w.skills`; the retry step skips `st.ReloadRetry`, which no test reaches, so an implementation that skips the full list or retries again passes.
**Fix:** Extend the table's timeout handling so a row first moves to the retry step (a `missing` report for the first turn end) and then lets the retry turn time out, asserting that only the retry skill is skipped with `cause=timeout`, that no further load is typed and that the pointer follows.

### [NIT] A reset assertion keeps a literal step value

**Location:** `internal/orchengine/watcher_test.go:904`
**Issue:** `st.ReloadStep != 0` checks the reset to the skills step by literal, where the diff introduced `ReloadStepSkills` for exactly this value and its sibling assertions use it.
**Fix:** Compare with `ReloadStepSkills`.
