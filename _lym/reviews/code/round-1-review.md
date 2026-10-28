---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttleengine/claudeengine/prepare_test.go:293
    summary: comments on the append-system-prompt tests and constants still describe a file holding only the deny notice and the creel trust sentence
    fix: In prepare_test.go reword line 293 to say the file holds the deny notice, the creel trust sentence and the batching sentence, space-separated, with the notice left out when empty, and reword the testtiming:keep reason on line 295 to say the file equals the notice, the trust sentence and the batching sentence. In settings.go reword the noticeTextForbiddenChars comment at line 403 so it says the notice shares one line of the append-system-prompt file with the sentences that follow it, not only the creel trust sentence.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: contracts/stencils/burler/burler-step-1-explore.md:41
    summary: the new reading sentence follows the fork_reading marker, so in a fork round it lands after the closing go-straight-on-to-step-2 instruction and its check them clashes with judge nothing while you read
    fix: Move the sentence to sit before the {{.tool_use_rules}}{{.fork_reading}} line, keeping that line's markers adjacent and unchanged so markers_test.go and the burlerengine template test still place fork_reading directly after the tool-use rules, and word it so a fork round does not read it as licence to judge, for example "Read the files of your focus, or of the whole target when no focus is set, in one message." Update the burlerengine template_test.go comment that says the fork reading sits at the end of the stencil.
  - id: F3
    severity: LOW
    class: consistency
    origin: handler
    location: docs/overview.md:480
    summary: the creel module entry says the append-system-prompt.txt file carries only the creel sentence, though it now also carries the batching sentence
    fix: In the creel entry of docs/overview.md line 480, reword the sentence to say the file carries the creel sentence and one further sentence that asks for batched calls, or point at the shuttle entry that now describes the file's contents, so the two entries agree.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/webster/webster-body-implementer.md:54
    summary: run the card's gate once sits in step 2 while step 3 runs the gate as several separate foreground calls, so once can be read as one call
    fix: In step 2 of webster-body-implementer.md reword the end of the added sentence to run the card's gate, as step 3 lists it, after the card's last edit rather than after each edit, so it neither reads as a single call nor repeats step 3's gate rule.
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: contracts/stencils/darn/darn-template-write.md:74
    summary: Step 4 reads every file before the first edit though the step repeats for each commit, and item 2 has a mismatched subject
    fix: In item 1 of Step 4 change before the first edit to before the first edit of each commit. In item 2 reword the tail so the subject is consistent, for example "it computes its scope from your changes; never run a raw test command or a module-wide run."
---

## Findings

### [LOW] Stale comments on the append-system-prompt file's content

**Location:** `internal/shuttleengine/claudeengine/prepare_test.go:293`, `prepare_test.go:295`, `settings.go:403`
**Issue:** The diff adds the batching sentence to the file and updates the code, `doc.go`, the file header of `prepare_test.go` and the resume-line comment, but three comments still name only the creel trust sentence. The test comment at line 293 states the old content outright, the `testtiming:keep` reason at line 295 says the test pins the notice and the trust sentence, and the `settings.go` comment says the notice shares its line with the creel sentence alone.

### [LOW] Reading sentence follows the fork_reading marker

**Location:** `contracts/stencils/burler/burler-step-1-explore.md:41`
**Issue:** In a solo round the line reads as the step's last instruction. In a fork round `fork_reading` ends with the reading manifest and the go-straight-on-to-step-2 instruction, and the new sentence comes after it, so it reads as an afterthought and its then-check-them clause contradicts the same text's judge-nothing-while-you-read rule. The marker line itself is unchanged, so `markers_test.go` still holds. The `TestTemplate_OptionalDirectives` comment in `internal/burlerengine/template_test.go` that says the fork reading sits at the end of the stencil is no longer accurate once the sentence follows it, and moving the sentence ahead of the marker line restores that.

### [LOW] The creel module entry names only the creel sentence

**Location:** `docs/overview.md:480`
**Issue:** The shuttle entry at line 413 was updated, but the creel entry still says every Claude session gets the sentence that tells it to follow `[creel]` mail through one `append-system-prompt.txt` file. The file's contents are now two sentences, so this line is incomplete. The focus asked for exactly this kind of leftover; the affected-set lens searched for the file name and the sentence name and did not reach this line.

### [LOW] Webster sentence sets the gate once in step 2

**Location:** `contracts/stencils/webster/webster-body-implementer.md:54`
**Issue:** The sentence matches D4 in intent and keeps the per-card gate and commit in steps 3 and 4. Its wording, run the card's gate once, sits in step 2 ahead of step 3's list of gate steps, each its own foreground Bash call, so a reader can take once as a single call. It also repeats a gate instruction before the gate rule that carries it.

### [NIT] Darn Step 4 wording

**Location:** `contracts/stencils/darn/darn-template-write.md:74`
**Issue:** The gather sentence says before the first edit, but Step 4 runs per commit and the darn writer commits in small steps, so later commits cannot gather before the first edit of the session. Item 2 joins two clauses whose subjects differ, the gate and the way it is run. The item keeps the green-commit rule, the never-a-raw-test-command rule and the message wording.

## Rejected

- Pin the batching sentence's literal text in a test (code-tests lens): the engine's one definition is the constant, the code-correctness lens confirmed it is byte-equal to the board entry's sentence, and a second literal in a test only duplicates the constant; the creel sentence is tested the same way.
- No test of the `init` line-break check for the batching sentence (code-tests lens, raised as not a finding): a package-load panic over a constant cannot be reached without editing the constant, and the creel check it extends had none.
- The commit trailer names Claude Opus 5.5 where the session reminder names Sonnet 5.5 (code-affected-set lens, raised as not a finding): not a repository rule, and the reminder governs the session's own commits, not this one.
- Stencil changes for the Discussion, Plan, twill and landing writers: each matches D2's placement and uses message; the landing sentence sits in the `## What to read` section as D2 asks.
- The burler fixer's two sentences: they split D2's one sentence across the Validate and Fix steps, which fits the sequence and drops no existing fixer rule.
