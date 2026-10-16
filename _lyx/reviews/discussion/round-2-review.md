---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:61-64
    summary: D5's Bound names only webster's plan-level gate as a consumer of the verify section, but Publish and Finalize rerun the same section after a parent merge, where the card and batch gates it cites as the bound never run
    fix: In D5's Bound, after the sentence naming webster's plan-level verify gate, state that the `## verify:` section is also the chained command Publish and Finalize run after a parent merge, so a tier an entry calls compiled rather than run also goes unrun at landing; then state what bounds that landing skip (the card and batch gates ran the untagged and `-tags integration` tests before the merge only, and `publish_verify` runs only what landing config names), instead of presenting the card and batch gates as bounding the whole skip.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:28-32
    summary: D1 settles that a scope-adding, non-overturning finding is planned without recording, but the rework stencil text it prescribes names only the overturning case, while the plan stencil the rework writer applies exactly as written routes every late scope addition through the parent
    fix: In D1's Decision, after the sentence naming the one case of the route, add that the rework stencil also states a finding that overturns no Decision is not a late scope addition under the plan stencil's route and is planned directly, licensing its card's scope; this is the rework stencil's own exception to the plan stencil (its Step 1 already says it applies the plan stencil except where the rework prompt says otherwise), so it points at the route without restating it. Add a matching clause to the rework bullet of Acceptance criteria.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:78-83
    summary: D6 drops the implementer's compile-only sentence for the tmux and llm tags although it names only Go-coded mechanics the Out list keeps, so with an empty list the implementer is told llm-tier runs go through lyx gate test, and D7's claim that the guards keep the neutral rule safe is false for the implementer
    fix: In D6's implementer bullet, keep the sentence that a test under the `tmux` or `llm` tag that no gate step names is compiled, never run, through `lyx gate test --tags <tag> <packages> -- -run '^$'`, as a guard-and-gate-describing sentence beside the raw-go guard sentence, on the same footing D6's Rationale gives the guard sentences; the list's compile-only rule then adds tiers beyond those two. Drop the implementer clause from Open risks, and add that sentence to the Acceptance criterion's exception for sentences describing Go-coded guards and gates.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:73-74
    summary: D6's plan bullet does not say whether the plan stencil's clause requiring each package's hermetic build-tagged tests stays, and Open risks names the vet compile steps but not the integration tier the neutral rule may leave out of the plan section
    fix: In D6's plan bullet, state whether the `## verify:` paragraph's requirement to run each targeted package's tests including its hermetic build-tagged tests stays as a language-neutral rule; if it leaves, add to the first Open risks bullet that plan writers here may omit `go test -tags integration` from the plan section, which Publish and Finalize then do not run after a parent merge.
---

### [MEDIUM] D5's Bound misses the landing consumer of the verify section

**Location:** `_lyx/discussion/decision-record.md:61-64`
**Issue:** The Bound says an entry calling a tier compiled rather than run lets the plan's `## verify:` section leave that tier unrun, and that this skip is bounded by the Guard classification and by the card and batch gates.
`contracts/specs/loom-plan-spec.md`'s `## verify model` says landing is the section's second consumer: Publish and Finalize run the same chained command whenever the tree differs from the last verified tree, and the section "must cover what a parent merge can break".
The card gate (`cardGateCommand` in `internal/websterengine/cardgate.go`, untagged) and the batch gate (`renderBatchGate`, `--tags integration`) run inside webster only, before any Merge-In, so they bound nothing after a parent merge.
The Guard classification is accurate (`configengine.CategoryGuard` is "a key that switches off or hollows a guard, a review or an approval gate") and does bound who can set the key.
On the focus question: the Bound states the webster-side skip and its bounds correctly, but understates what the skip reaches.

### [MEDIUM] D1's non-overturning scope-adding finding has no stencil sentence

**Location:** `_lyx/discussion/decision-record.md:28-32`
**Issue:** D1 line 32 settles the decision: a finding that adds scope without overturning a Decision is planned with no recording, and the Plan-Review rubric bullet licenses it.
The prescribed rework stencil text, though, names only the overturning case as its use of the plan stencil's route.
`loom-template-rework.md`'s Step 1 tells the writer to apply the plan stencil "exactly as written, except where this prompt says otherwise", and `loom-template-plan.md:34-37` says any "scope addition that arrives after the Discussion" is planned only once the parent records it, with a stop when no parent is recorded.
A scope-adding finding is such an addition, so a writer reading both prompts asks the parent for it, and on a decline D1's whole-rework stop halts the run over a finding D1 meant to be planned directly.
The Plan-Review rubric accepts either outcome, so nothing downstream catches the divergence.
On the focus question: D1 settles that the whole rework stops on a decline or with no parent; the scope-adding case is settled as a decision but not carried into the rendered rework text.

### [MEDIUM] D6 drops the implementer's tmux/llm compile-only sentence

**Location:** `_lyx/discussion/decision-record.md:78-83`
**Issue:** `contracts/stencils/webster/webster-body-implementer.md:83-84` says a `tmux` or `llm` test no gate step names is compiled through `lyx gate test --tags <tag> <packages> -- -run '^$'`.
D6 moves that rule into the list, so with an empty list the implementer reads only that a module-wide, `tmux`-tier or `llm`-tier run "goes through `lyx gate test`".
That text agrees with the raw-go row in `internal/shellguard/rows.go`, which refuses raw `go test` with a `tmux` or `llm` tag, but `internal/gatecli` refuses no tag, so the implementer is pointed at running `llm` tests, which spawn a real LLM.
D7's Rationale says the guards keep the neutral rule safe for a Go hub; no guard stops this run, and Open risks records it as a gap instead.
The sentence names only `lyx gate test` and the guard's own tag pair, mechanics the Out list keeps Go-coded in every hub, so D6's own Rationale for keeping guard-describing sentences covers it, and keeping it closes the gap rather than leaving it to the window before the operator sets the value.

### [LOW] The neutral plan rule's build-tagged clause is unstated

**Location:** `_lyx/discussion/decision-record.md:73-74`
**Issue:** `loom-template-plan.md:201` requires the section to run each package's tests "including its hermetic build-tagged tests (for example `-tags integration`)".
D6 lists what leaves the paragraph (the order, the `llm` and `tmux` rules, the `go -C` form) and the neutral fallback, but not whether this clause stays.
If it leaves, a plan written under the neutral rule here may run only `go test`, and the Open risks bullet, which names only the vet compile steps, misses that the integration tier then goes unrun at Publish and Finalize.
