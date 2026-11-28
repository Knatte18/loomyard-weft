---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: internal/darnengine/spec_test.go#measurementEntryRule
    summary: The new measurementEntryRule assertions pin static stencil prose, which PATTERN-test-speed forbids, and they cannot tell the two fragments apart because both carry the sentence.
    fix: "Delete the `measurementEntryRule` constant and its doc comment at internal/darnengine/spec_test.go:85-86, and remove `measurementEntryRule` from the `wantInPrompt` lists of the rows \"first spawn renders the prior-work token as none\" (line 121) and \"a cold fixer on the verify gate has the writer and its subagents run no gate\" (line 149). Add no replacement assertion: review guards the wording."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:45
    summary: With no parent, the rule has the writer skip the entry's whole deliverable and still hand off an ordinary change description, and its instruction to name the gate calls there contradicts Step 5's ban on naming lym, a gate or a tier.
    fix: "In both darn-fragment-make-change-gated.md:45 and darn-fragment-make-change.md:27, replace `With no parent, run none of them and name them in the change description.` with a stop path in the template's own form: with no parent, or when the post is refused and no strand answers, run none of them; write the change description, your one output file, as a stop report in the final-summary spec's form naming the measurements the entry asks for and, as the way forward, a parent decision on the calls and their limits; create the stop marker; and end your turn with no commit. Word the stop report's content as the measurements and their scope, never as gate or tier calls."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: contracts/stencils/darn/darn-fragment-make-change.md:12
    summary: In the ungated list, item 2 still hands each subagent the full gate rule for its own package, and a subagent sees none of the session, so a writer that delegates a measurement entry's per-package work bypasses the new parent-first rule.
    fix: "In darn-fragment-make-change.md, keep a measurement entry's gate calls in the writer's own session: add to the item 2 line that lists the work kept in this session (line 13) the gate calls of an entry whose deliverable is gate measurements, and state in the new item 3 rule (lines 23-27) that no subagent runs them."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/darn/darn-fragment-make-change.md:23
    summary: "`falls outside these rules` and `before any gate call for it` do not say whether an entry that also changes code or commits its results keeps the ordinary per-commit gate run, so a mixed entry has two readings."
    fix: "In both fragments' new rule (darn-fragment-make-change.md:23-24 and darn-fragment-make-change-gated.md:41-42), name what the parent-first rule covers: the measurement calls alone. In the ungated fragment add that the per-commit gate run of item 3 still follows each commit of the entry's code or results; in the gated fragment, that the verify gate still tests the committed tree."
---

### [BLOCKING] Test assertions pin a stencil's static prose

**Location:** `internal/darnengine/spec_test.go:85-86`, `:121`, `:149`

**Issue:** `PATTERN-test-speed` says no test pins a stencil's static prose.
A stencil test may pin only a marker set, a Fill round-trip, a Go-filled marker's position, a value Go fills, reads or names, a fragment Go chooses, or a rule across a stencil family (`pattern/PATTERN-test-speed.md:284-288`).
`measurementEntryRule` is the opening words of a sentence that both fragments carry word for word (`darn-fragment-make-change.md:23`, `darn-fragment-make-change-gated.md:41`).
Go neither fills nor names it.
Because both fragments hold it, it cannot show which fragment Go chose; `ungatedTestStep` and `gatedNoGateStep` already pin that choice in the same rows.
The rubric's Test economy bullet names a new assertion on static stencil prose as a finding, and a violation of a listed constraint is BLOCKING.

### [MEDIUM] The no-parent path drops the deliverable and contradicts Step 5

**Location:** `contracts/stencils/darn/darn-fragment-make-change-gated.md:45`, `contracts/stencils/darn/darn-fragment-make-change.md:27`

**Issue:** For an entry whose deliverable is the measurements, "run none of them" means the writer makes nothing the entry asks for.
The rule neither stops the writer nor sets a stop marker, so Step 5 writes an ordinary change description and the run hands off and can land as if it were done.
Every other path in the template where the writer cannot do the entry without a parent writes a stop report and creates the stop marker (`darn-template-write.md:60-63`, `:103`).
"name them in the change description" asks the writer to list the planned `lym gate test --tier …` calls.
Step 5 forbids the change description to name lym, a gate or a tier (`darn-template-write.md:78`).
The ungated list's older no-parent clause names skipped tests (`darn-fragment-make-change.md:17`), which are repository objects, so it does not have this conflict.
The parent directive sends a question that no strand answers to "where your role's instructions send a stop it cannot settle" (`contracts/stencils/parent/parent-directive-parent.md`).
The new rule names no such stop, so a refused or unanswered post has nowhere to go either.

### [MEDIUM] Ungated subagents can run measurement calls the rule bounds

**Location:** `contracts/stencils/darn/darn-fragment-make-change.md:12`

**Issue:** Item 2 of the ungated list tells the writer to give each subagent the gate rule "for the subagent to run on its own package", and it says a subagent sees none of the session.
The new parent-first rule is addressed to the writer alone.
A writer that splits a per-package measurement series across subagents passes them the unbounded per-package gate rule, which is the incident's loop shape again.
The gated list does not have the gap: its item 2 tells every subagent to run no `lym gate test` (`darn-fragment-make-change-gated.md:23`).

### [LOW] A mixed entry's per-commit gate run is ambiguous

**Location:** `contracts/stencils/darn/darn-fragment-make-change.md:23`

**Issue:** The rule takes the entry as a whole "outside these rules" and requires the parent's answer "before any gate call for it".
An entry that commits result files or also changes code still needs item 3's per-commit gate run (`darn-fragment-make-change.md:14`).
Read literally, that run waits on the parent too.
Read loosely, the measurement part slips back under the per-commit rule, which sets no bound.
The gated fragment's placement leaves the verify gate's own run clear, but the same "any gate call" wording governs its trigger.

## Rejected

- code-tests finding 2 (the `measurementEntryRule` assertions are redundant as coverage of fragment selection): the same defect as F1, which removes it.
- code-affected-set F1 (naming gate calls in the change description contradicts Step 5): the same defect as code-correctness F2, and both are kept as F2, whose fix replaces the sentence.

## Holistic notes

- Scope: the diff takes the entry's second option, having the writer ask its parent before the calls, in both Step 4 lists. Every other hunk is a header comment, doc comment or test the rule forces.
- Doc comments: `stencils.go`'s two fragment docs, the `DarnTable` doc and both fragment headers state the rule for both lists, and none of them contradicts the fragments.
- Change description: its claims match the diff. A `two-way` door with a `darn` blast radius holds, because the diff changes stencil wording, comments and one test, and no record format, config meaning, data or push.
- Commits: neither commit in the range carries a `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:` trailer, and none is needed. The directive declares no PATTERN changes.
