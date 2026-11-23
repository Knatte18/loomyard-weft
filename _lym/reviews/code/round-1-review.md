---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shellguard/rows.go:390
    origin: lens:code-affected-set
    summary: "The tmux-script steer's way forward omits --tier, which board item 1 requires, so a call in its form runs tier0 and skips the tmux-tier test; it also points at a single-test form only the darn writer's instructions give"
    fix: "Rewrite steerTmuxScriptDeny so its way forward is self-sufficient and names --tier, keeping it free of quotes and backslashes: after the outlives-every-memory-cap clause, say to write a tmux-tier test in the repository and run that one test through lym gate test with --tier naming the tier the hub runs tmux tests in, the test's package, and after -- a filter that selects it. Drop the clause in the form your instructions give for a single test. noticeTmuxScriptDeny follows from the steer. Regenerate internal/shuttleengine/claudeengine/testdata/settings-golden.json and rerun the engine tests that compare the steer."
  - id: F2
    severity: MEDIUM
    class: design
    location: contracts/stencils/darn/darn-fragment-make-change.md:18
    origin: lens:code-affected-set
    summary: The ungated writer is told to run its tmux-tier test through lym gate test with its package and a filter but no --tier, so the gate's default tier0 run skips the test and the writer reads a fact from a test that never ran
    fix: "In Step 3's new lines, change the run sentence to: and run that one test through `lym gate test --tier <that tier>` with its package and, after `--`, a filter that selects it; then check that the gate's report shows the test ran, not skipped."
  - id: F3
    severity: MEDIUM
    class: scope
    location: contracts/stencils/burler/burler-template-fixer.md
    origin: lens:code-scope
    summary: "Board item 3 asks the writer and fixer stencils to carry the tool-learning and no-background rules, but only the darn writer fragments and the cold gate fixer frame changed; the burler, webster card and landing fixers carry neither rule"
    fix: "Add item 3's two rules, worded as in gatefix-frame-cold.md's External tools and processes section, to each writer and fixer stencil whose agent edits repository code: burler/burler-template-fixer.md, webster/webster-fragment-card-fixer.md and landing/landing-template-fixer.md at least, and any other code-editing writer or fixer stencil a grep for lym gate test under contracts/stencils turns up. Where the stencil lets its agent run lym gate test, give the single-test form with --tier, its package and a filter after --."
  - id: F4
    severity: MEDIUM
    class: decision
    location: internal/shuttleengine/claudeengine/settings.go:378
    origin: lens:code-scope
    summary: "Board item 1 asks for the refusal in every run agent's Bash, but the row is installed only when the spec is not interactive, so run sessions that can launch interactive (Discussion seat, PR session, drift judge, raddle, seat chair) keep raw tmux and script; the narrowing is the writer's own call, recorded nowhere"
    fix: "Leave the code in place and defer this finding to the run's parent: recording or refusing the autonomous-only narrowing with lym loom decision add is the parent's call and lies outside the fixer's write surface."
  - id: F5
    severity: LOW
    class: decision
    location: contracts/stencils/darn/darn-fragment-make-change-gated.md:30
    origin: lens:code-scope
    summary: "Board item 2 leaves how narrow the tmux-tier opening is to a Discussion that never happened; the writer settled it in stencil prose alone, and nothing in lym gate test stops a run agent from running the whole tmux tier"
    fix: "Leave the code in place and defer this finding to the run's parent, who records the prose-only narrowing with lym loom decision add or asks for a gate-side check."
  - id: F6
    severity: LOW
    class: consistency
    location: internal/darnengine/spec.go:89
    origin: lens:code-affected-set
    summary: The DarnTable doc comment still says the gated writer has one exception to running no gate, while the gated fragment now names two
    fix: "Replace the line with: The exceptions are the writer's single measuring run of one test class, for a value only a test run shows or a fact about how an external tool behaves."
---

### [BLOCKING] The tmux-script steer's way forward cannot be followed as written

**Location:** `internal/shellguard/rows.go:390`
**Issue:** Board item 1 says the refusal names the way forward: write a tmux-tier test and run it through `lym gate test --tier`.
The steer says to run the test through `lym gate test`, naming its package and a filter, "in the form your instructions give for a single test".
It never names `--tier`, and `lym gate test` with no tier flag runs tier0 only (`internal/gatecli/doc.go:29`).
A tmux-tier test calls `tiergate.Require` and skips under tier0 (`internal/testkit/tiergate/tiergate.go:70`).
The row is installed for every autonomous role, but only the darn writer fragments give a single-test form.
Reviewers, shed drivers and the burler, webster, landing and cold fixers get a way forward that points at nothing.
The incident happened because the safe path was closed or unclear, so a way forward that silently runs no test is the defect this row exists to remove.

### [MEDIUM] The ungated writer's tmux-tier run form runs tier0 only

**Location:** `contracts/stencils/darn/darn-fragment-make-change.md:18`
**Issue:** The new lines tell the writer to write the test in the tmux tier, then run it through `lym gate test` with its package and a filter.
They give no `--tier`, so the run is tier0 and the test is skipped.
The gated fragment avoids this because its measuring call is `--gate darn-write`, whose key lists tiers 0, 1 and 2 in this hub.

### [MEDIUM] Item 3 reaches only the darn writer and the cold gate fixer

**Location:** `contracts/stencils/burler/burler-template-fixer.md`
**Issue:** The board entry asks "the writer and fixer stencils" to say: learn how an external tool behaves from a gated test, and start no background or detached process.
The diff changes the two darn fragments and `gatefix-frame-cold.md`.
A grep of `burler-template-fixer.md`, `webster-fragment-card-fixer.md`, `landing-template-fixer.md` and the twill writer forks finds neither rule.
These fixers edit code, and the steer now refuses their raw `tmux`, so they need the same guidance.

### [MEDIUM] The row skips interactive run sessions, which the entry did not license

**Location:** `internal/shuttleengine/claudeengine/settings.go:378`
**Issue:** The change description reasons that the sampler's operator-attended agent is told to start `tmux -L`, so the row hangs on `!Interactive`.
That also leaves raw `tmux` and `script` open to run sessions that launch interactive: the Discussion seat with `discussion_interactive`, the non-autonomous PR session, an interactive drift judge, the interactive raddle and the seat chair.
The entry says "every run agent's Bash".
The record has no Discussion and no added entry for this narrowing, so it is the parent's to record or refuse.

### [LOW] The tmux-tier opening is narrowed in prose only

**Location:** `contracts/stencils/darn/darn-fragment-make-change-gated.md:30`
**Issue:** Item 2 asks that "a run cannot start the whole tmux tier", with Discussion settling how.
No gate code refused a run agent's tmux-tier `lym gate test` before this diff, and none does after it.
The stencil's measuring call names one package and one class, and the change description says no gate-side check was added.
The writer's own call stands until the parent records it.

### [LOW] Stale one-exception doc comment in darnengine

**Location:** `internal/darnengine/spec.go:89`
**Issue:** The comment says "The one exception is the writer's single measuring run of one test class for a value only a test run shows."
The gated fragment now opens Step 3 with "There are two exceptions."

## Rejected

- Code-correctness lens: no findings. Its run of the pattern over the incident command, path forms, quoted text, substitution and nested shells, and its trace of both engines' switch maps, agree with my reading.
- Code-tests lens: no findings. Every behaviour has a pinning row, and the changed counts in the engine tests tighten, never loosen, what was asserted at the base.
- Code-scope F2, the steer's missing `--tier`, is merged into F1.
- `Root-Cause:` on a7937a620 names the missing table row, which the diff adds; no finding.
