---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:15-19
    summary: The gated fragment spells a gate scope form and Go test-command arguments, which PATTERN-test-speed forbids a stencil to name and which assume a Go hub
    fix: >-
      Take the board entry's second route, which the constraint allows: the writer asks its parent for the one run.
      In contracts/stencils/darn/darn-fragment-make-change-gated.md, restore item 3's opening line to "3. Run no `lym gate test` yourself."
      and replace the four added lines (16-19) with an instruction to ask the parent.
      When the task starts from a reported test failure, before diagnosing it, the writer sends `lym creel post --parent --expect-reply "<text>"`
      asking its parent to run that test once on the writer's base through `lym gate test`, then ends its turn to wait.
      When the parent reports that the test passes, the failure no longer reproduces on the writer's base:
      the entry is one that landed work overtook, and Step 3 settles it.
      The new text names no package, target, `--` argument or test flag.
      Rewrite the comments that describe the exception to say the writer runs no gate and asks its parent for the reported failure's single run:
      the fragment's header comment (line 3), contracts/stencils/stencils.go:55, internal/darnengine/spec.go:67-68 and 79-81, and internal/darnengine/doc.go:81-82.
      In _lym/landing/summary.md, replace the design decision "The writer runs the test itself rather than asking its parent" with the parent route,
      and give PATTERN-test-speed's no-scope-form rule as the reason.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:16-19
    summary: A run of the reported test that names no tier and no exact test can skip it or match nothing and still pass, and the stencil reads any pass as the failure no longer reproducing
    fix: >-
      In the reproduce instruction (as F1 rewrites it), have the writer's request name the test exactly and the tier the test belongs to.
      Have the writer read a pass as the failure no longer reproducing only when the run's report shows the named test ran,
      not when the test was skipped for another tier or the run matched no test.
      Without such a report, the writer diagnoses the failure as usual.
  - id: F3
    severity: BLOCKING
    class: consistency
    origin: lens:code-tests
    location: internal/darnengine/spec_test.go:77
    summary: The new gatedReproduceRun constant pins the gated stencil's static prose, which PATTERN-test-speed forbids; fragment choice is already pinned by gatedNoGateStep and ungatedTestStep
    fix: >-
      In internal/darnengine/spec_test.go, delete the constant gatedReproduceRun (line 77) and its uses in the first row's wantNotInPrompt (line 104)
      and the gated row's wantInPrompt (line 112).
      Set gatedNoGateStep (line 76) to the item-3 opening line the fragment carries after F1, "3. Run no `lym gate test` yourself.",
      and restore the gated row's name (line 109) to "a cold fixer on the verify gate has the writer and its subagents run no gate".
      In internal/loomcli/wiring_darn_test.go:266, set gatedNoGateStep to that same line.
  - id: F4
    severity: LOW
    class: consistency
    origin: handler
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:15-19
    summary: The reproduce check sits in Step 4's item 3, after item 1's edits, yet it must come before any diagnosis and routes back to Step 3's before-any-code check
    fix: >-
      Move the reproduce instruction (as F1 and F2 word it) out of item 3, which keeps only its "Run no `lym gate test` yourself." line
      and its two verify-gate lines.
      Make it the opening sentences of item 1, ahead of "Edit the repository to make the change.", without renumbering the list,
      so the writer asks before its first edit or diagnosis.
---

### [BLOCKING] The stencil names a gate scope form and Go test flags

**Location:** `contracts/stencils/darn/darn-fragment-make-change-gated.md:15-19`

**Issue:** Item 3 tells the writer to "name its package and add `-- -run <TestName>`".
`PATTERN-test-speed` is a listed constraint.
Its background says, at `pattern/PATTERN-test-speed.md:226`: "A site or stencil names a gate key, never a tier list, scope form, test command or target list".
A package target is a scope form, and `-- -run` is test-command arguments.
`-run` is also `go test`'s flag.
The darn fragment renders on any hub, and a .NET or Python hub's tier command would receive a flag it does not take.
The answer key offers two routes: run the test through `lym gate test`, or have the prompt tell the agent to ask its parent for that single run.
The first route, as written, conflicts with the constraint, and the constraint wins.
The second route complies.
A gated writer always has a parent: `writerFixerOn` turns the fixer on only for a non-empty pair slug (`internal/loomcli/gatefixer.go:158-161`).
The change description rejects the parent route only on round-trip cost, and cost does not outweigh the constraint.
No `Pattern-Departure:` trailer is in the range, and no PATTERN change is declared.

### [BLOCKING] A run that skips the reported test reads as a pass

**Location:** `contracts/stencils/darn/darn-fragment-make-change-gated.md:16-19`

**Issue:** The instruction gives no tier.
Without `--tier`, `--upto` or `--gate`, `lym gate test` runs tier0 alone (`internal/gatecli/cli.go:75`).
A tier1 test then reaches `tiergate.Require` and calls `t.Skipf` (`internal/testkit/tiergate/tiergate.go:70-71`).
`go test` exits 0, and a run carrying a `-run` filter is exempt from the empty-tier check (`internal/gatecli/tierrun.go:419-421`).
The stencil then says "When it passes, the failure no longer reproduces", so the writer stops with a false overtaken entry.
The incident's own package, `internal/shedverbs`, holds tier1 tests.
A mistyped name, or an unanchored `-run` that matches another test, gives the same false reading.
Under F1's parent route the same gap holds unless the request names the tier and the exact test, and the writer checks that the test ran.

### [BLOCKING] TestDarnSpec pins the stencil's new prose

**Location:** `internal/darnengine/spec_test.go:77`, used at `:104` and `:112`

**Issue:** `gatedReproduceRun` holds a sentence of the gated fragment's wording.
The test asserts that the sentence is present in the gated prompt and absent in the ungated one.
`PATTERN-test-speed`'s background, `pattern/PATTERN-test-speed.md:248-252`, says "No test pins a stencil's static prose".
A stencil test may pin a fragment Go chooses, but `gatedNoGateStep` and `ungatedTestStep` already tell the two fragments apart.
The new assertion adds no signal about Go's choice.
This is also the rubric's test-economy case "a new assertion on the static prose of a stencil".
The `TestWire_DarnRecipe` constant change was a needed sync: the old string ended `yourself.` and would no longer match.

### [LOW] The reproduce check comes after the edit step

**Location:** `contracts/stencils/darn/darn-fragment-make-change-gated.md:15-19`

**Issue:** The board entry asks for the run "before diagnosing".
The instruction sits in item 3, after item 1's "Edit the repository to make the change".
On a pass it sends the writer back to Step 3, which the template titles "Check the entry before any code" (`darn-template-write.md:43`).
"Before you diagnose it" limits the damage, but a writer that follows the list in order reaches the check after its edits.

## Focus verdicts

- Item 2's subagent rule is unchanged, and the added text opens no further gate runs. The loophole question is not a defect.
- The template does have a Step 3, and its outdated-entry stop settles an overtaken entry (`darn-template-write.md:43-56`). The pointer is not dangling.
- The `-- -run` form matches `lym gate test`'s argument handling (`internal/gatecli/cli.go:85,108`), but the item omits the tier; see F2. A form that names no `--gate` key hands no failure to a cold fixer (`internal/gatecli/cardfixer.go:81-84`).
- The `DarnTable` doc's "They act only on…" now trails the exception sentence. The comments also attribute the exception to "the writer and its subagents" (`spec.go:67-68`). F1's fix rewrites all of these comments.
- No other test or stencil holds the old Step 3 string.

## Rejected

- code-affected-set F3, the stale "drops its gate runs" wording at `internal/loomcli/darnwiring.go:279` and `internal/loomcli/wiring_darn_test.go:262`: under F1's fix the gated writer again runs no gate, so that wording stays correct, and changing it now would make it wrong.
- code-correctness finding 2, the unanchored `-run`: F1's fix removes the line, and F2's fix requires the request to name the test exactly.
- code-correctness, the out-of-lens note that the `GateFixed` comment grants the exception to subagents: folded into F1, whose fix rewrites that comment.
