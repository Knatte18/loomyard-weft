---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    location: internal/loomshed/planwrite.go:116
    summary: The NewPlanDirRotator result sentence breaks after the colon, leaving the fragment starting with empty when nothing moved on a line of its own
  - id: F2
    severity: LOW
    location: internal/loomshed/planwrite_test.go:241
    summary: The seedPriorPlanStencils doc and the inline comment in MissingStencilWithPriorFilesIsAnErrorAndMovesNothing are fixed-column wrapped mid-phrase
  - id: F3
    severity: LOW
    location: internal/shedadapters/singlellm_test.go:977
    summary: The PrepareFreshSpawnErrorNeitherArchivesNorSpawns and PrepareFreshSpawnAmendment test docs are fixed-column wrapped mid-phrase
  - id: F4
    severity: LOW
    location: internal/shedrecipe/entries_planwrite_test.go:162
    summary: The TestPlanWriteEntry_CallAppendsPriorPlanBlock doc is fixed-column wrapped across three lines
  - id: F5
    severity: LOW
    location: internal/shuttleengine/claudeengine/prepare_test.go:155
    summary: The TestPrepare_AppendSystemPromptMatchesDenyNotice doc is fixed-column wrapped across three lines
  - id: F6
    severity: LOW
    location: internal/loomrecipe/fixture_test.go:74
    summary: The edited seedBouncerStencils first sentence breaks mid-phrase on its changed lines
  - id: F7
    severity: LOW
    location: internal/shuttleengine/claudeengine/command_test.go:265
    summary: The fork_mode_real_notice_pwsh inline comment breaks before a participial phrase, not at an independent-clause boundary
  - id: F8
    severity: NIT
    location: internal/shuttleengine/claudeengine/command.go:77
    summary: Several long new sentences keep an independent-clause boundary unbroken on one line (command.go:77, settings.go:36, priorplan.go:15, planwrite.go:120 and :122)
---

The range is `git diff $(git merge-base main HEAD)..HEAD`: `product.parent` is `main` in `_lyx/shed/plan-write-respawn/status.json`, and the merge base is `6b41264db`.
HEAD is `e29b05465`, the same commit the round-1 judge verified, so no rewrap commit exists yet and every comment the judge named is unchanged.
`go build ./...`, `go vet -tags smoke ./...` and `go test -tags integration ./...` all pass on HEAD.

The plan's cards carry only `Edit` and `Create` groups, so the per-card mechanical check has no `Rename` AST script or `Delete` `assert-no-callers` to run.
The build and the integration suite are what the Edit and Create groups require, and both pass.

The findings below apply CONSTRAINTS.md's Comment Line-Break Convention, which covers Go comments "doc and inline alike", test files included, and governs new and changed comments only.
I swept every `//` line the diff adds, not only the lines named in the focus directive.
Every other added comment breaks only at sentence ends or independent-clause boundaries.

### [LOW] NewPlanDirRotator result sentence breaks before a fragment

**Location:** internal/loomshed/planwrite.go:116

**Issue:**
The sentence "The closure's string result is the prompt amendment SingleLLMProducer appends to the respawned session's prompt: empty when nothing moved, otherwise …, so the new session knows where the prior plan went." breaks after the colon.
"empty when nothing moved, otherwise a blank line followed by …" has no subject or verb of its own, so line 117 is a fragment, not an independent clause.
The break before ", so the new session knows" is correct.

**Fix:**
Join lines 116 and 117 and keep the break before "so the new session knows".

### [LOW] planwrite_test.go new comments are fixed-column wrapped

**Location:** internal/loomshed/planwrite_test.go:241

**Issue:**
The `seedPriorPlanStencils` doc breaks "into a temp / stencils directory", mid-phrase.
The inline comment at line 394, which the round-1 F1 fix added, breaks "leaves the plan where a retry / finds it again", mid-clause.

**Fix:**
Put the doc sentence on one line.
In the inline comment, break only before "-- otherwise the retry would rotate nothing", or keep the sentence on one line.

### [LOW] singlellm_test.go new test docs are fixed-column wrapped

**Location:** internal/shedadapters/singlellm_test.go:977

**Issue:**
The `TestSingleLLMProducer_PrepareFreshSpawnErrorNeitherArchivesNorSpawns` doc breaks "the failure posture: a / preparation".
The `TestSingleLLMProducer_PrepareFreshSpawnAmendment` doc at line 1008 breaks "the text a preparation / returns".
Both breaks fall mid-phrase.

**Fix:**
Rewrap each sentence with a line break only at its independent-clause boundary: before "and nothing downstream of it runs" in the first, and none in the second, whose ", and to no other spec" is a fragment.

### [LOW] TestPlanWriteEntry_CallAppendsPriorPlanBlock doc is fixed-column wrapped

**Location:** internal/shedrecipe/entries_planwrite_test.go:162

**Issue:**
The one-sentence doc breaks "and a / minimal prior-plan stencil" and "a spec / whose prompt", both mid-phrase.
The ", then asserts" continuation is a compound predicate, not an independent clause.

**Fix:**
Put the sentence on one line.

### [LOW] TestPrepare_AppendSystemPromptMatchesDenyNotice doc is fixed-column wrapped

**Location:** internal/shuttleengine/claudeengine/prepare_test.go:155

**Issue:**
The one-sentence doc breaks "of the deny / inputs" and "if and only if / buildDenyNotice", both mid-phrase.

**Fix:**
Put the sentence on one line.

### [LOW] seedBouncerStencils edited sentence breaks mid-phrase

**Location:** internal/loomrecipe/fixture_test.go:74

**Issue:**
The diff changes lines 74, 77 and 78 of the doc's first sentence, which is therefore in scope as a whole.
Its changed lines break "or / Webster-Review", "plus / loom-template-prior-plan" and "each seeded / from", and the sentence ends mid-line on line 79, where the next sentence starts.
The sentence has no independent-clause boundary: what follows the colon lists the stencils.

**Fix:**
Put the first sentence on one line, and start line 79 at "shedadapters.NewBouncer".
Leave the unchanged later sentences' legacy wrap alone.

### [LOW] fork_mode_real_notice_pwsh inline comment breaks before a participial phrase

**Location:** internal/shuttleengine/claudeengine/command_test.go:265

**Issue:**
The comment is one verbless sentence: "A real assembled notice, whose ':' and ';' must stay inside the one quoted argument, riding inside the fork-mode env wrap rather than after it."
It breaks before "riding …", a participial phrase, not an independent clause.

**Fix:**
Put the comment on one line.

### [NIT] Long new sentences keep an independent-clause boundary unbroken

**Location:** internal/shuttleengine/claudeengine/command.go:77

**Issue:**
The convention defers to scribe:prose, which says a long sentence also breaks at an internal independent-clause boundary.
These new sentences keep one or more such boundaries on a single line:

- `command.go:77`: ", so the session is told which tools are denied" and "; an empty notice leaves the line unchanged".
- `settings.go:36`: "; it must hold for every session that receives it".
- `priorplan.go:15`: ", so an empty archiveDir or an empty movedFiles fails with the unfilled-marker error".
- `planwrite.go:120`: ", and a render failure is returned as the closure's error".
- `planwrite.go:122`: ", so the next attempt renders the block again".

The round-1 fixes applied exactly this break elsewhere, for example after the semicolon in `loomengine/plan.go:16` and before "so the preparation can tell" in `singlellm.go`, so leaving these unbroken is inconsistent within the same diff.

**Fix:**
Break each of these sentences at the named boundaries, and change nothing else on those lines.
