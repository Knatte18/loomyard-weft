---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/darn/darn-fragment-make-change.md:40
    summary: "The closed-scope way forward (a board note, then lym loom resume) re-spawns a writer whose Step 4 rule still says to run none of the calls and stop with no allowing entry, so the resumed no-parent writer stops again on every resume; the same holds in darn-fragment-make-change-gated.md:56."
    fix: "In both contracts/stencils/darn/darn-fragment-make-change.md (after line 40) and contracts/stencils/darn/darn-fragment-make-change-gated.md (after line 56), add one sentence to the measurement rule's no-parent branch, before \"Then create the stop marker\" is reached on a resumed spawn: on a task a Code-Review round has run on, a `Decision for {{.slug}}:` board note that says the scope is closed and files the measurements for a new task settles the stop, so the writer runs none of the calls, makes what the record asks without them, and names the note in the change description's design decisions. The fragments are filled by stencil.Fill with only subagent_model and measure_call (internal/darnengine/spec.go:188), so {{.slug}} is not available there: write `Decision for <slug>:` with the task's slug in words, as darnStopWayForward does in internal/loomshed/darnwrite.go:32. Keep the wording identical in both fragments."
  - id: F2
    severity: LOW
    class: decision
    origin: handler
    location: internal/darnengine/spec_test.go#measureNoParentWayForward
    summary: "The two new TestDarnSpec assertions, measureNoSubagent and measureNoParentWayForward, check static stencil prose that both fragments now carry, so they tell neither fragment apart and pin no behaviour; the rubric's test-economy bullet counts a new assertion on static stencil prose as a finding."
    fix: "The entry is labelled bug, so the red-first gate needs a test that fails on the merge base, and the change is stencil prose alone. Two choices, the parent's to make: (a) accept the prose assertions as the entry's red-first seam and leave the code as it is; or (b) remove both constants (internal/darnengine/spec_test.go lines 85-91), drop them from the two wantInPrompt rows (lines 126 and 154), and have the parent record a `No test seam` entry with lym loom decision add, which a closed scope still takes for a bug entry (internal/loomcli/decision.go:1028-1036). The fixer cannot record that entry itself."
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/darn/darn-fragment-make-change.md:39
    summary: "\"after which you run them under that entry\" sits inside the text the writer is told to write as the stop report's way forward, so the report's reader, the parent or operator, can read \"you\" as themselves; darn-fragment-make-change-gated.md:55 carries the same clause, and Step 3's way forward in darn-template-write.md:61-62 has no such clause."
    fix: "In both contracts/stencils/darn/darn-fragment-make-change.md:39 and contracts/stencils/darn/darn-fragment-make-change-gated.md:55, replace \"then `lym loom resume`, after which you run them under that entry;\" with \"then `lym loom resume`, after which the resumed writer runs them under that entry;\". Line 54 of the gated fragment and line 38 of the ungated one stay unchanged, so measureNoParentWayForward in internal/darnengine/spec_test.go still matches."
---

### [MEDIUM] Closed-scope way forward loops the resumed writer

**Location:** `contracts/stencils/darn/darn-fragment-make-change.md:40`, and the same sentence at `contracts/stencils/darn/darn-fragment-make-change-gated.md:56`.

**Issue:** I traced the open-scope path and it works. `lym loom decision add` from the operator's unnamed shell passes `decisionCallerRefusal` (`internal/loomcli/decision.go:772`). `lym loom resume` on the halted run spawns a Go driver (`internal/loomcli/resume.go:141-156`). Darn-Write then removes both stop markers and spawns the writer again (`internal/loomshed/darnwrite.go:166-170`), and the writer finds the entry and runs the calls.

The closed-scope branch does not work. Its trigger, "a task a Code-Review round has run on", matches the `reviewRan` condition under which `decision add` refuses (`internal/loomrecipe/scopeline.go:42`), so the board note is the right step. But after the note and `lym loom resume`, the re-spawned writer reads only "With no such entry, run none of them and stop" (lines 35 and 51). It stops again, and `darnWrite.Call` halts Stuck again (`darnwrite.go:189-210`). Every further resume repeats this.

The template's Step 3 has the sentence that ends this loop: a `Decision for <slug>:` note that says the scope is closed settles the point (`darn-template-write.md:53-55`). That sentence covers only the outdated-entry stop, and its no-parent bullet (line 60) says a note never settles a stop while the scope is open. The own-entry drop route has its own clause (line 104). The measurement rule has neither. So the change description's claim that the new wording "mirrors the way forward the darn template's Step 3 gives" holds for the stop report's wording but not for what the resumed writer does.

### [LOW] New TestDarnSpec assertions pin static stencil prose

**Location:** `internal/darnengine/spec_test.go#measureNoParentWayForward`, with `measureNoSubagent` and the two rows at lines 126 and 154.

**Issue:** Both fragments now carry both lines, so neither assertion tells the gated row from the ungated one. Fragment selection is already pinned by `ungatedTestStep`, `ungatedSubagentGate` and `gatedNoGateStep`. What the new assertions check is the wording of the stencil, which the rubric's test-economy bullet names as a finding.

They exist to meet the bug label's red-first requirement, and the change description says they fail on the merge base. The rubric's alternative, a `No test seam` entry, is a parent call that a closed scope still accepts for a bug entry. The fixer cannot record it, so this is a decision.

### [NIT] "you run them" in the stop report's way forward

**Location:** `contracts/stencils/darn/darn-fragment-make-change.md:39`, and the same clause at `contracts/stencils/darn/darn-fragment-make-change-gated.md:55`.

**Issue:** The writer is told to write this way forward into a stop report that the parent or operator reads. Copied as it stands, "after which you run them" addresses that reader, who never runs the calls. Elsewhere the template tells the writer that it never runs `lym loom decision add` itself, and that verb is for the reader. "You" in the same sentence then means the writer, so the two senses of "you" collide.

## Rejected

- Gated subagent bar incomplete: item 2 tells each subagent to run no `lym gate test`, and line 46 keeps the measurement calls in this session, matching the ungated fragment's lines 15 and 30. No path remains for a subagent to make a measurement call.
- Merge-danger door two-way: the diff changes only stencil prose and a test. A revert undoes it, and it touches no record format, config meaning or data. The blast radius, darn, covers what the change can break.
- Doc comments left unchanged: `contracts/stencils/stencils.go:55,64` and `internal/darnengine/spec.go:95` state only that the writer asks before the measurement calls, which still holds, as the change description claims.
- The closed-scope trigger wording: "on a task a Code-Review round has run on" matches `scopeClosedAt`'s `reviewRan` test. The Finalize exclusion cannot apply at Darn-Write.
