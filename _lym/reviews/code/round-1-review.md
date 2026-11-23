---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/boardengine/store.go:508
    summary: validateWrite still refuses a dependency on a note and refuses making a depended-on entry a note, against the added decision that a task or a note may depend on any entry, a note included.
    fix: "In Store.validateWrite (internal/boardengine/store.go:508-524), delete the loop that refuses a dependency whose target is a note and the `if incoming.Kind == KindNote` block that refuses making a depended-on entry a note. Keep the dangling, isolated and cycle checks; findCycle already walks a note's DependsOn through Dependencies.waitsOn, so a cycle through a note stays refused. In internal/boardengine/dependency.go, make Dependencies.metBelow judge a dependency on a note as not met whatever its status, so a task waiting on a note waits until the note is promoted and lands or is removed. In the same change, drop the depends-note refusal in internal/raddleparser/check.go (CheckDependsNote, around lines 338 and 387) and its row in contracts/specs/raddle-proposal-spec.md:99, which go stale with it. In internal/boardengine/store_test.go TestKindDependencyRules, add rows that a task and a note each take a note as a dependency, that a cycle through a note is refused, that a depended-on task can be demoted to a note, and that the dependent's Pending lists a note dependency even when that note carries status done."
  - id: F2
    severity: BLOCKING
    class: scope
    origin: lens:code-scope
    location: internal/boardengine/store.go:777
    summary: RemoveTask leaves the removed slug in every other entry's depends_on, against the added decision that removing an entry strips it as prune does.
    fix: "In Store.RemoveTask (internal/boardengine/store.go:777), after splicing the entry out, strip its slug from the DependsOn of every remaining entry, as Prune does for done entries. Board.RemoveTask, the `lym board remove` verb and fronttask's RemoveEntry go through it. In MergeTasks (store.go around line 1315), which also calls RemoveTask, keep a dependency on a slug the merge upserts again. Update the RemoveTask doc comment to state the strip, and add a test in internal/boardengine/store_test.go that removes an entry a task and a note both depend on and asserts both lose it while keeping their other dependencies."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: plugins/ly/skills/board/SKILL.md:258
    summary: The skill line saying a note cannot be depended on, and the boardengine doc's any entry that lands, contradict the added decision that a note may be depended on.
    fix: "In plugins/ly/skills/board/SKILL.md, replace the line \"A note cannot be depended on, so a relation between notes is a sentence in a body.\" with a line saying a note may depend on another note, set with `lym board set-deps`, and that a task waiting on a note waits until the note becomes a task and lands or is removed. In internal/boardengine/doc.go, replace \"A task or a note may depend on any entry that lands, a project included.\" with \"A task or a note may depend on any other entry: a task, a note or a project.\" and add that a dependency on a note is met only once the note is promoted and lands, and that removing an entry strips it from every depends_on. Rewrite the summary's Design decisions bullet in _lym/landing/summary.md once F1 lands."
  - id: F4
    severity: LOW
    class: consistency
    origin: assist:assist
    location: internal/boardengine/siblings_test.go:297
    summary: The new row set-deps on a note waiting on what an open task waits on asserts the same behavior as the row note takes a dependency and makes no sibling.
    fix: "Delete the TestWriteSiblings row named \"set-deps on a note waiting on what an open task waits on\" (internal/boardengine/siblings_test.go:297-300). Keep the row \"note takes a dependency and makes no sibling\", which seeds the same D, an open task Y waiting on D with label board and a note with label board, runs setDeps on the note and asserts no sibling and the stored dependency."
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/store.go:38
    summary: The BriefTask doc comment names Layer and HasProposal as computed at read time but not the new Pending.
    fix: "Change the BriefTask doc comment at internal/boardengine/store.go:38 to \"BriefTask is an entry's enriched read-only view, with Layer, Pending and HasProposal computed at read time.\""
---

### [BLOCKING] A note still cannot be depended on

**Location:** internal/boardengine/store.go:508
**Issue:** The record's added decision reads: "A task or a note may depend on any other entry: a task, a note or a project. A cycle is refused, as for tasks today. A task that depends on a note cannot start until that note has become a task and landed, or is removed, and the board shows it waiting."
The diff keeps the refusal of a note target (store.go:508-513) and the refusal to make a depended-on entry a note (store.go:514-524).
The change description says outright "A note still cannot be depended on … the board entry asked for nothing more", which ignores the added entry.
No `Pattern-Departure:` trailer explains the difference.
The waiting semantics are only half there: Dependencies.metBelow (dependency.go:51) counts any entry with status done as met, so a note marked done would release its dependent before it was ever promoted.
The raddle check `CheckDependsNote` and its spec encode the same refusal and must move with it.

### [BLOCKING] Removing an entry leaves it in other entries' depends_on

**Location:** internal/boardengine/store.go:777
**Issue:** The added decision reads: "Removing an entry strips it from every other entry's depends_on, as prune does for a done entry."
RemoveTask only splices the slice, and Board.RemoveTask and the `remove` verb add nothing.
The leftover slug makes the two views the diff adds disagree:
Dependencies.met counts a missing slug as met, so `pending` and the `list --text` after column drop it,
while the README's unmetDependencies (render.go:750) counts a missing slug as unmet and keeps it in After.
The next validated write of that entry is then refused as a dangling dependency.

### [MEDIUM] The board skill and package doc state the opposite of the added decision

**Location:** plugins/ly/skills/board/SKILL.md:258
**Issue:** The diff adds "A note cannot be depended on, so a relation between notes is a sentence in a body." to the `ly:board` skill.
The decision's rationale is the operator's "a note may depend on a note, such as one idea building on another".
internal/boardengine/doc.go:152 says "any entry that lands", which also excludes a note, since a note never lands.

### [LOW] Two sibling rows pin the same behavior

**Location:** internal/boardengine/siblings_test.go:297
**Issue:** Both rows seed D, an open task waiting on D and a note that shares the label `board`, run `setDeps` on the note with D, and assert no sibling and the stored dependency.
Only the slugs and the seed order differ, and neither affects the behavior under test.

### [NIT] BriefTask doc omits Pending

**Location:** internal/boardengine/store.go:38
**Issue:** The type's doc lists the fields computed at read time, and Pending is now one of them.

## Rejected

- code-tests F1, no test of the `list --text` after column: the compact listing is output a human or agent reads, with no machine parser (code-affected-set found none), so the testing rule exempts its layout; the data behind it, `Pending`, is pinned by TestListTasksBriefLayerAndProposal.
- code-tests F2 and mutation survivors render.go:682, 703 and 704: the README's note table is human-read output, which the testing rule exempts from layout tests.
- code-correctness F1, pending and README disagree on a missing slug: the same defect as F2, whose fix removes the case; merged there.
- code-scope findings 2, 4, 5 and 6: the same defects as F1 and F3; merged.
- code-affected-set F2, raddleparser check and spec stale: folded into F1's fix, since they change only with it.
- assist F1 and F2: the same defects as F1 and F2; merged, keeping the assistant's note on a done note into F1.
- assist F4, missing t.Parallel in the new subtests: PATTERN-test-speed is enforced by a test, which the gate runs, and the surrounding subtests of TestPromote and TestKindDependencyRules follow the same form; a gate-decided rule is not this perch's subject.
