```yaml
lens: code-affected-set
outcome: reported
```

My lens was code-affected-set: what the diff's changes force outside the lines it touches. I found one MEDIUM finding (F1) and one NIT (F2).

**The affected set, per change**

1. **Way-forward wording changed in both Step 4 fragments, ungated and gated.** Everything that reads or describes it:
   - `internal/loomshed/darnwrite.go:32` (`darnStopWayForward`, the Go suffix added to the stop reason). It still agrees: the same `lym loom decision add`, then `lym loom resume`, and a board note once the scope is closed. Covered.
   - `contracts/stencils/darn/darn-template-write.md:54,60-62` (Step 3, the outdated-entry stop). The wording mirrors it. But how a resumed writer acts differs; see F1. Not covered.
   - `contracts/stencils/darn/darn-template-write.md:88-93,104` (scope correction and own-entry drop). The own-entry drop has a `Decision for &lt;slug&gt;:` note clause that settles the stop once the scope is closed; the measurement rule has none. See F1.
   - `internal/loomcli/decision.go:110,378-409` and `internal/loomrecipe/scopeline.go:19-43`. "A task a Code-Review round has run on" matches when the verb refuses: the `reviewRan` check, before Finalize. Covered.
   - `internal/loomcli/decision.go:765-805` (`decisionCallerRefusal`). With no parent, the operator's own unnamed shell may run `add`, so the named step can be taken. Covered.
   - `internal/loomshed/darnwrite.go:151-211` and `internal/loomcli/resume.go`. A Stuck stop resumes and re-spawns Darn-Write, which removes the stop markers first. Covered.
   - Doc comments at `contracts/stencils/stencils.go:53-66` and `internal/darnengine/spec.go:85-96` say only "asks its parent before", which still holds. Covered, as the change description claims.
   - `internal/darnengine/spec_test.go` rows. Covered; how its test economy applies is the code-tests lens's call.
   - The orch role docs and specs carry no measurement-rule text, so nothing there goes stale.

2. **Subagent sentence added to the gated fragment.**
   - It matches the ungated fragment's line 30 word for word.
   - The gated header comment (lines 1-9), the `stencils.go:60-66` doc and the `spec.go` doc all say "the writer and its subagents run no gate", which still holds. Covered.

3. **The two new constants in `spec_test.go`.** Only the two rows of `TestDarnSpec` use them. Covered.

**Other checks**

- **Declared PATTERN changes:** the directive declares none, and none are needed.
- **Repository rules:**
  - PATTERN-stencil-ownership holds: both fragments are edited in their own stencil files.
  - PATTERN-refusal-way-forward: the new stop way forward names a step the operator can run. But on a closed scope that step does not reach an end; see F1.
- **Commit message:**
  - The `Root-Cause:` trailer names both stencil defects the diff touches.
  - The claim "as the ungated fragment already did" holds at the merge base: the diff leaves that line unchanged.
  - There is no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed.

**Findings**

- **F1. Severity MEDIUM, class design.**
  - **Location:** `contracts/stencils/darn/darn-fragment-make-change.md:40` and `contracts/stencils/darn/darn-fragment-make-change-gated.md:56`.
  - **Summary:** on a closed scope, the way forward is a board note, then `lym loom resume`. But neither fragment, nor the template, lets the resumed writer take a `Decision for &lt;slug&gt;:` note as settling the measurement stop. The re-spawned writer finds no allowing `Added after Discussion` entry and stops again, on every resume.
  - **Evidence:**
    - The fragments' lines 51/35 say "With no such entry, run none of them and stop."
    - `darn-template-write.md:54` limits the note-settles-it clause to Step 3's outdated-entry stop.
    - `darn-template-write.md:104` gives the same clause only to the own-entry drop.
    - `internal/loomshed/darnwrite.go:166-170,209` re-spawns the writer and halts Stuck again whenever the marker is written.
  - **Fix:** add to the measurement rule's no-parent branch the clause Step 3 has. On a task a Code-Review round has run on, a `Decision for &lt;slug&gt;:` note that says the scope is closed and files the measurements for a new task settles the stop. The writer then makes what the record asks without the measurements, and names the note in the design decisions.

- **F2. Severity NIT, class consistency.**
  - **Location:** the same lines, 39 and 55 ("after which you run them under that entry").
  - **Summary:** the clause sits inside the text the writer is told to write as the stop report's way forward. A reader of the report, the parent or operator, can take "you" to mean themselves, not the writer.
  - **Evidence:** `darn-fragment-make-change.md:38-39`. Step 3's way forward at `darn-template-write.md:61-62` names only the verbs and carries no such clause.

**Tool calls**

1. grep for the changed wording across the code, contracts and docs. My context had no list of who reads it.
2. Read `stencils.go:50-70`, `spec.go:85-100`, and searched docs and PATTERN for darn measurement text. These are the doc comments the change description claims are untouched, and my context did not hold them.
3. grep of the orch role, specs and stencils for measurement text and `Decision for` notes. I needed to confirm where a note settles a stop.
