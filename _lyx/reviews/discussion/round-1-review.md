---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D5, D13 and the first Constraint
    summary: Retagged tier0 test files drop out of impactset's and planglyph's package loads, which list under integration,tmux,llm only, so narrowed gates silently skip tests, during the transition and after deploy
    fix: In D5 add that every Go package load that lists test files under the old tag set moves to the tier tags (tier0,tier1,tier2,tier3), naming impactset's reverse-dependency graph and guard-package listing and the plan gate's type load, so a diff scope sees tier0 test imports. In D13 replace the claim that the dual-tag commands are right for old and new trees alike with the stated gap and its bound, that the installed lyx's narrowed gates (--base and computed) miss a package reached only through a tier0 test import, and that this run's whole gates (Webster's final --all and Merge-In's landing --all) are what catch it, or make retagged test files carry the transitional term so the installed graph still sees them. Add an acceptance criterion that a diff scope includes a package reached only through a tier0 test file's import.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md D2 (diff) and D4
    summary: D2 takes as its base any whole-green entry for the tier, without requiring the current fingerprint, empty args and the same module directory, so a filtered or stale whole green becomes a base and breaks the chain's soundness
    fix: In D2 change the base to the newest ancestor commit whose tree the green record holds whole-green for that tier under the current tier fingerprint, with empty args and the same module directory. In D4 state how many whole-green commits per tier pruning keeps. Extend the D2 acceptance criterion with a whole entry recorded under -- -run X, or under a changed tier command, which is never taken as a base.
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D8 and the acceptance criterion on resume, relaunch and goto
    summary: D8's relaunch and bypass rules do not match the code paths, since a tackler or arbiter relaunch without a target leaves the entered row named, a targeted one moves the run as Goto does, the arbiter's relaunch is not named, and the orch and Go driver also call goto
    fix: In D8 state the entered-row rule per path. A relaunch without a target re-calls the entered row without evaluation, as a resume does. A relaunch with a target, tackler's and arbiter's alike, leaves the target not entered, so it is evaluated. Every goto through the shed goto verb, whoever calls it, names its target as entered. Change the bound from the operator's goto to the goto verb as the one bypass, and change the acceptance criterion from a tackler relaunch to a tackler or arbiter relaunch with a goto target evaluates it, while one without a target re-calls the entered row.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D8
    summary: D8 does not place entry-gate evaluation relative to the step's pause_before check, so a pause_before naming a gated-off row in a skip chain never fires, and a row entered then paused resumes without re-evaluating a setting changed during the pause
    fix: In D8 add the rule. A pause_before naming a row pauses the run before that row's entry gate is evaluated, so a skip chain stops at a paused row. The row is marked entered only once its gate holds after the pause check. Add an acceptance criterion that pause_before on a gated row halts before evaluation and that a setting changed during the pause counts.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D1, D3 and D12
    summary: Darn's tree_scope_verify command, which runs in place of the gate at Darn-Write, Code-Burler-Fix and darn Finalize, is not addressed, so those sites keep an operator-written command with targets that cannot name a gate key, contrary to the declared PATTERN-test-tiers text
    fix: Add a Decision on darn.yaml's tree_scope_verify. Either retire it, because the diff scope of darn-write, code-burler-fix and finalize replaces it, with reconcile pruning the key, or keep it with a stated way for its runs to carry the site's gate key, for example a --gate key with the configured tiers and the tree's targets as the scope, and a stated exception in the PATTERN-test-tiers change. Add the darn tree-scoped site to D12's table either way.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 Bound
    summary: The carry's bound omits that the green record is a plain file under the worktree's .lyx, which any agent session can write with no standing deny, so a forged whole entry makes Merge-In and Finalize carry without running, and the fingerprint ignores the inherited environment
    fix: In D4's Bound name what a forged or hand-edited record lets through, every tier of every later gate on that tree Merge-In and Finalize included, and what bounds it, whether a deny on writes to the gate directory, a record agents cannot write, or an explicit acceptance as for the slot variable in gatecli's doc. Also state that inherited environment variables outside the tier's env are not in the fingerprint.
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D7
    summary: D7's zero-tests failure trips on any -run filter after --, since a filter that matches nothing in a package runs zero tests there though the package holds tier tests, so a green filtered run fails
    fix: In D7 exempt a run whose args carry -run (or -list or -bench), or count a package as vacuous only when its tests were not filtered, and state the bound. Add an acceptance criterion that a filtered run over packages the filter misses does not fail.
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lyx/discussion/decision-record.md D13 and Open risks
    summary: The discovered-count check on the first retagging card, the open risk's only catch for missing hub tier commands, names no actor that reads the count or a mechanism that fails on zero
    fix: In D13 replace the check with a mechanical precondition that has an owner, for example the first retagging card's Verify line runs a gate over one retagged package with a -run naming a known test and fails when the tier's command lacks its tag, or the plan's first card's Verify line checks that the hub's tier commands carry -tags tier0. Update the open risk to name that mechanism.
  - id: F9
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lyx/discussion/decision-record.md D4 with D2
    summary: D4 does not say whether a run that carries part of a tier and passes the remainder records the tree as whole-green, so darn's Finalize all after Darn-Write's diff leaves no whole entry and D2 falls back to the merge base
    fix: In D4 add that a tier whose carried part and run remainder together cover every target writes a whole entry for the tree.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D3
    summary: D3 says the writers' verify gate keeps a latest-pass lint base, but today's darn form lints from the merge base on every attempt, so the change to darn's lint range is presented as unchanged
    fix: In D3 state that the darn form's lint base moves from the merge base on every attempt to the latest recorded pass, else the merge base, or keep the merge base for the darn form and say so.
  - id: F11
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lyx/discussion/decision-record.md D9 and D10
    summary: Moving batten's switches onto a bare operand drops the Command producer's refusal of a non-bool switch, so a value such as the string no opens the window, and the record states no bound
    fix: In D9 or D10 either make recipe build or evaluation refuse a non-bool value for a bare operand whose setting is declared bool, or state the downgrade as the bound.
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lyx/discussion/decision-record.md Acceptance criteria, the carry-breaking criterion
    summary: The criterion says args other than a -run filter break the carry, implying a user's -run filter keeps it, while D4 lets an empty-args entry cover only the guard step's own -run and needs an exact match otherwise
    fix: Reword the criterion to say any args other than the guard step's own -run filter break the carry unless they match the entry exactly.
  - id: F13
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lyx/discussion/decision-record.md D10, the --window line
    summary: The --window force lives in the inner run's hidden --open-terminal flag, which forces Open-Window, not in --window itself, so the value D10 places on --window has to come from the inner run's flag
    fix: In D10 say that the inner run's --open-terminal flag, which --window passes on, is what battencli's Env.Setting answers batten.open_window true for.
  - id: F14
    severity: NIT
    class: consistency
    origin: handler
    location: _lyx/discussion/decision-record.md D5 Rationale, second sentence
    summary: The sentence on the rename widening the edit and the measured 20 s gain is deliberation narrative with no requirement for the plan writer
    fix: Move the sentence starting The rename adds the already-tagged files to support-log.md's Interview turn 4 and delete it from D5.
---

### [BLOCKING] Retagged tier0 test files vanish from impact derivation

**Location:** `_lyx/discussion/decision-record.md` D5, D13 and the first Constraint

**Issue:** `impactset.DeriveTargets` builds the reverse-dependency graph, test imports included, with `go list -tags integration,tmux,llm` (`internal/impactset/derive.go:59`, `internal/impactset/doc.go:18`).
`ModuleGuardTests` lists packages the same way (`internal/impactset/guards.go:102`), and so does the plan gate's type load (`internal/planglyph/typesload.go`).
A test file tagged only `tier0` is invisible to all three.
If package A's only test importing B is `//go:build tier0`, a diff changing B leaves A out of every narrowed gate, which passes without running A's test.
During this run and on `main` until deploy, the installed `lyx` runs every narrowed gate that way.
D5 moves only `cmd/testtiming` and `planparser` to the tier tags, so the same miss stays in the `diff` scope D2 builds after deploy.
D13 handles only the guard scan, so its claim that the change is "right for old and new trees alike" holds for the tier commands and nothing else.
D7 cannot catch it, because A is never a target.

### [MEDIUM] D2's base search ignores the args, fingerprint and module that D4 keys on

**Location:** `_lyx/discussion/decision-record.md` D2 and D4

**Issue:** Suppose `lyx gate test --tier 0 --all -- -run TestFoo` passes at commit C.
D4 then writes a whole entry for C.
D2 picks "the newest commit … whose tree the green record holds whole-green", with no condition on args, so a later `diff` run takes C as its base.
That breaks D2's own claim that "a narrowed green never becomes a base".
A whole entry recorded under a since-changed tier command, or for a nested module run through `-C`, opens the same gap.
D4's pruning keeps "recent" whole-green commits without saying how many.

### [MEDIUM] D8's relaunch and bypass rules disagree with the relaunch paths

**Location:** `_lyx/discussion/decision-record.md` D8 and its resume, relaunch and goto acceptance criterion

**Issue:** `shedengine.Relaunch` without a target writes the status unchanged, and with a target "moves the run as Goto does" (`internal/shedengine/goto.go:187-191`).
So a tackler resume with no goto keeps the entered row and is never evaluated, which contradicts the acceptance criterion "a tackler relaunch evaluates it".
A targeted relaunch that shares Goto's entered-row write would be a second bypass.
Loom's arbiter also relaunches through `Relaunch` (`internal/loomcli/arbiterconsume.go`), and D8 does not name it.
The orch agent and the Go driver's goto seam (`internal/battencli/drive.go`) also reach goto, so "the operator's goto is the one bypass" is not true as written.

### [MEDIUM] Entry-gate evaluation is not ordered against pause_before

**Location:** `_lyx/discussion/decision-record.md` D8

**Issue:** The step checks `pause_before` against `CurrentProducer` before the call (`internal/shedengine/run.go:240-248`).
D8 evaluates the gate when the run enters a row and chains skips within one step.
A `pause_before` naming a gated-off row therefore never fires and stays recorded.
A row that is entered and then paused resumes without re-evaluation, so a setting the operator changes during that pause has no effect.

### [MEDIUM] Darn's tree-scoped verify command is left out

**Location:** `_lyx/discussion/decision-record.md` D1, D3 and D12

**Issue:** For a diff wholly under `darn.yaml`'s trees, Darn-Write, darn's Code-Burler-Fix and darn's Finalize run the `tree_scope_verify` command in place of the gate (`internal/loomcli/darnwiring.go:31-53`, `pattern/PATTERN-verified-tree.md`).
That command is an operator-written `lyx gate test` call with explicit targets.
D3 refuses `--gate` together with targets, so these sites can never name their key.
Their records stay keyless, the `darn-write` and `finalize` tiers and scope don't apply to them, and the declared `PATTERN-test-tiers` text ("a site names its gate key, never tiers or scope") is broken by an existing site the record never mentions.

### [MEDIUM] The carry's bound omits a forged record and the inherited environment

**Location:** `_lyx/discussion/decision-record.md` D4 Bound

**Issue:** The green record is a plain file under the worktree's never-tracked `.lyx` gate directory.
Agent sessions run in that worktree, and the standing denies in `internal/shuttleengine/claudeengine/settings.go` cover only Bash rows, Agent and AskUserQuestion, so nothing stops a session from writing it.
A forged whole entry for HEAD's tree makes every later gate on that tree carry, Merge-In and Finalize included.
The bound names a flaky test, the toolchain and the machine, but not who can write the record.
Compare `internal/gatecli/doc.go`, which states its bound for a forged slot variable.
The tier also inherits this process's environment, and nothing in that environment is fingerprinted.

### [MEDIUM] D7 fails filtered runs

**Location:** `_lyx/discussion/decision-record.md` D7

**Issue:** Words after `--` still reach a `--gate` run and a manual run.
A `-- -run X` over a package where X matches nothing runs zero tests there while the package holds tier tests, so D7 fails a green run.
D7 states no exemption or bound for filtered args.

### [MEDIUM] The discovered-count check has no owner

**Location:** `_lyx/discussion/decision-record.md` D13 and Open risks

**Issue:** "The first retagging card's gate must report the retagged packages' tests as discovered" names no one who reads the count and nothing that fails on zero.
Webster's card gate passes on its exit code, and the implementer stencil has no such rule.
The open risk calls this check "the only catch" if the orch has not changed the hub's tier commands.
The hub's `gate.yaml` today still has the untagged `tier0` command and `covers: [0]`, so the risk is live.

### [LOW] A tier carried in part and run for the rest writes no whole entry

**Location:** `_lyx/discussion/decision-record.md` D4 with D2

**Issue:** Darn's Finalize `all` carries Darn-Write's `diff` packages and runs the remainder, so the tree is fully green.
D4 says an entry is "either whole or the green package set" and does not say this run records the tree as whole.
D2 then finds no whole-green ancestor and falls back to the merge base, in the very case D4's rationale cites.

### [LOW] D3 calls a change to darn's lint base unchanged

**Location:** `_lyx/discussion/decision-record.md` D3

**Issue:** D3 says the writers' gate "keeps" a latest-pass lint base, else the merge base.
The darn form today lints from the merge base on every attempt (`internal/loomshed/verifygate.go`, `darnForm` branch), so darn's lint range does change.

### [LOW] The bare operand drops the non-bool switch refusal

**Location:** `_lyx/discussion/decision-record.md` D9 and D10

**Issue:** `CommandProducer` refuses a switch that is not a bool (`internal/shedadapters/command.go:98-100`).
D9's bare operand holds for any set, non-zero value, so `open_window: "no"` opens the window, and no bound states the downgrade.

### [LOW] The carry criterion contradicts D4 on -run

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria

**Issue:** "Args other than a `-run` filter break the carry" implies that a requested `-run` keeps it.
D4 lets an empty-args entry cover only the guard step's own filter, and needs an exact match for any other args.

### [LOW] The --window force is on the inner run's flag

**Location:** `_lyx/discussion/decision-record.md` D10

**Issue:** `--window` opens a window running `lyx batten run --open-terminal`, and only that inner run forces Open-Window (`internal/battencli/cli.go:122`, `internal/battencli/wire.go:749-750`).

### [NIT] Deliberation narrative in D5's rationale

**Location:** `_lyx/discussion/decision-record.md` D5 Rationale

**Issue:** The cost-against-gain sentence carries no requirement for the plan writer, and support-log.md's turn 4 already records the choice.

## Rejected

- The D13 follow-up board entry asks for a change to this task's own board entry: rejected, because it files a separate entry that depends on this one, which the rubric allows.
- D6 lets markers into `tier0` and `tier1` without a bound: rejected, because D6 states both the allowed and the refused tiers, and a guard group skips only when its tier ran or carried whole.
- D1's `gate_key` has no bound: rejected, because D1 states that it selects only a key of the hub file.
- An untracked or ignored file can change a test's outcome on a clean worktree: rejected as too narrow to change the design; `verifytree.DirtyPaths` already counts untracked files that are not ignored.
