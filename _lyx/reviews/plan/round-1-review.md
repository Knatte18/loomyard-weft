---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/01-audience-and-index-renderer.md
    summary: Invocable counts a group's own persistent flags as local flags, so the burler and webster groups become runnable groups that no card annotates, and card 16's audience check fails on both
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/plan/06-fabric-form.md
    summary: card 6 trims only cleanup, remove and merge-in, but fabric clone's Long is about 4.2 kB, so card 16's 2000-byte cap fails; the form cards list their over-cap pages by hand and never measure them
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/00-overview.md
    summary: the plan neither carries nor reconciles the record's added Watcher index binary decision, whose rationale declines the launch-path plumbing that cards 13 and 14 build
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-fabric-form.md
    summary: cards 6 and 8 add per-package audience assertions that card 16's tree walk already pins for every command, while cards 4, 5, 7 and 9 rely on card 16 alone
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/01-audience-and-index-renderer.md
    summary: card 1 prints a runnable group only when its own annotation matches, while D2 shows a group whenever one leaf under it is in the audience, and the card leaves undefined what happens to children of a runnable group that does not print
---

### [BLOCKING] Persistent flags make `burler` and `webster` runnable groups that nothing annotates

**Location:** `_lyx/plan/01-audience-and-index-renderer.md` (`Invocable`), with `_lyx/plan/04-small-modules-form.md`, `_lyx/plan/08-webster-form.md` and `_lyx/plan/16-cli-form-guard.md`

**Issue:** Card 1 defines `Invocable` as true for a group with its own `RunE` and "a local non-hidden flag of its own".
Card 1's flag run is likewise built "from `LocalFlags` minus hidden ones".
Cobra's `LocalFlags()` includes the persistent flags declared on the command itself.
`internal/burlercli/cli.go:157-159` declares `--stencils-dir` and `--target-dir` as persistent flags on the `burler` group, and `internal/webstercli/cli.go:280-284` declares `--stencils-dir`, `--plan-dir` and `--target-dir` on the `webster` group.
Both groups also use `RunE: clihelp.GroupRunE` (`internal/burlercli/cli.go:153`, `internal/webstercli/cli.go:276`).
`lyx burler --help --json` and `lyx webster --help --json` both list these flags.
Under card 1's rule, both groups are therefore invocable.
Cards 4 and 8 set no audience on either group: card 4 annotates only burler's three leaves, and card 8 lists `webster`'s verbs only.
Card 16's audience check then fails "on an invocable command ... without the annotation" for `burler` and `webster`, so the plan's final card goes red.
Annotating them instead would make the index render `burler` and `webster` in leaf form, with flags that belong to their children, which is not what D3's runnable group (the `lyx config [module]` case) means.

**Fix:** In card 1, define `Invocable` and the leaf flag run over the command's non-persistent local flags (`LocalNonPersistentFlags()` minus hidden ones).
A group's persistent flags then stay with the children they configure, and `config`'s `--print`/`--set` still qualify.
Add a row to card 16's synthetic audience subtests: a `GroupRunE` group whose only flags are persistent is not invocable.

### [BLOCKING] Card 6 misses `fabric clone`, which is over the `Long` cap

**Location:** `_lyx/plan/06-fabric-form.md` ("`Long` texts over the cap are trimmed ...: `cleanup`, `remove` and `merge-in` today")

**Issue:** `fabric clone`'s `Long` in `internal/fabriccli/fabric.go` (from line 70) is about 4.2 kB.
`lyx fabric clone --help` prints 4,188 bytes before `Usage:`, more than twice D9's 2,000-byte cap.
Card 6 rewrites clone's `Use` but leaves it out of its trim list, so card 16's `Long` check fails on `fabric clone`.
I measured every command's help text against the cap; the other over-cap pages each fall in some card's trim list.
Those pages are fabric `cleanup`, `remove` and `merge-in`, loom `start` (and its root alias), `resume` and the `loom` group, and webster `reset`, `validate` and the `webster` group.
The defect is in the method: cards 4 to 9 name their over-cap pages by hand, as of "today", and no card measures them.
A hand-made list goes stale silently, which is how clone was missed.

**Fix:** Add `clone` to card 6's trim list, with its rationale moved to the new `internal/fabriccli/doc.go`.
Have each form card (4 to 9) define its trim set by measurement rather than by a list: every page in the package whose `Long` exceeds 2,000 bytes, measured for example by `lyx <path> --help --json | jq '.long|utf8bytelength'`.

### [MEDIUM] The "Watcher index binary" decision is neither carried nor reconciled

**Location:** `_lyx/plan/00-overview.md` (`## Shared Decisions`), `_lyx/plan/13-strand-lyx-bin.md`, `_lyx/plan/14-role-file-command-index.md`

**Issue:** The decision record has a later section, "Added after Discussion (parent, 2026-10-10): Watcher index binary".
It says the watcher renders the index from "its own recorded path", and that D4 claims equality with the orch pane's `LYX_BIN` only on the `StartRelaunch` branch.
It also says the `StartSpawnWatcher` and reed-resume mismatch is an accepted risk, because the mismatch "is not worth new launch-path plumbing in this task".
That rationale describes the design where the watcher's path can differ from the pane's.
That is the opposite of card 13's `Strand.LyxBin`, a new launch-path record, and of card 14's rule that the source "never runs the watcher's own executable".
Scope "In", D4's body and the acceptance criteria still describe the strand-recorded path, so the record contradicts itself.
The plan follows the other added decision ("Missing-binary bound at auto-reload") explicitly, citing it in card 14.
For this one it silently takes D4's original side.
Every Decision in the record must be carried by some card, or the plan must state how it reads a conflicting one.
As it stands, the "Watcher index binary" decision is carried nowhere.

**Fix:** Add a Shared Decision that names the conflict and records the reading the plan implements, with its reason.
For example: "its own recorded path" is read as the orch strand's recorded path, because Scope "In", D4 and the acceptance criteria all require reed to record it, so cards 13 and 14 stand.
If the plan instead adopts the added decision, drop card 13 and rewrite card 14's source to run the path the watcher records for itself, with the mismatch stated as a risk.

### [MEDIUM] Cards 6 and 8 duplicate card 16's audience check

**Location:** `_lyx/plan/06-fabric-form.md` (`TestCommand_VerbDeclarations` "gains an assertion that every verb carries an audience from the closed set"), `_lyx/plan/08-webster-form.md` (`TestCommand_LongStringsHaveNoStaleBatchLanguage` "gains the assertion that every verb carries an audience from the closed set")

**Issue:** Card 16's `TestCLIForm_EveryCommand` walks `newRoot()` and fails on any invocable command without an audience or with a value outside `clihelp.Audiences`.
That covers every fabric and webster verb.
The two per-package assertions therefore pin a behavior the plan's own guard already pins, which `PATTERN-test-economy` and the testing rule count as a defect ("a redundant test is a defect").
The plan is also inconsistent with itself: cards 4, 5, 7 and 9 change the same kind of annotations and state "coverage for the form is card 16's tree-walk test".
Card 8 also files an audience assertion under a test named for stale batch language, a behavior it has nothing to do with.
This is not BLOCKING: the rubric's test-economy item names an existing test, and here the covering test arrives later in the same plan.

**Fix:** Drop both assertions and state, as cards 4, 5, 7 and 9 do, that card 16's tree walk covers the annotations.

### [LOW] How a runnable group prints differs from D2's group rule

**Location:** `_lyx/plan/01-audience-and-index-renderer.md` ("An invocable command prints only when its annotation equals `audience`; a non-invocable group prints only when a command under it prints")

**Issue:** D2 says "a group appears only when at least one leaf under it is in the selected audience", and D3 calls runnable groups "leaf" too.
Card 1 prints a runnable group only on its own annotation.
It does not say what happens to that group's children when the group itself is not printed: dropped, or nested under a missing parent line.
The current tree has no such case (`config`, `help` and their children are all `operator`).
Card 1's test does not pin it, so a later reclassification would render undefined output.

**Fix:** In card 1, state the rule: a runnable group's line prints when its own annotation matches, or when one of its descendants prints.
If only descendants match, the line renders in group form.
Add that case as a row in `index_test.go`.

Verdicts on the focus entries with no finding:
- **Card order:** no card depends on a later card.
  Cards 4 to 8 use only card 1's constants, card 7 uses the `VerbText.Audience` field card 3 adds, card 14 reads the `StrandStatus.LyxBin` card 13 adds, and card 16 edits `newRoot`'s `Short` after card 2 edits its `Long`.
- **Fidelity:** D1 to D11, the intake-comments and strand-path items, and the docs and budget constraints are each carried by some card; F3 is the only gap.
  `upsert-issues-key` is licensed by D7's rule that `Use` equals the parser's declaration, and `upsertAllowedKeys` does accept `issues` (`internal/boardengine/store.go:348`).
  `index-note-annotation` is the mechanism D2 and D7 require.
  `help-mounted-through-set-help-command` is one of the two routes the Constraints allow.
- **Attack surface:** cards 2, 9, 12 and 14 each state their bound: the `--audience` closed-set refusal, `set-deps` "gains `id` addressing only", the whole-body pointer filter, and the watcher source's no-`PATH`, no-own-executable rule with a `Stat` check.
- **Glyphs and arrows:** no `Edit` card targets only Go file paths.
  Every re-signed member carries the arrow (`resolveLookup`, `decodeIntakePayload`, both `SetDeps`, `issueContent`, `inboxIssue`, `composePaneLaunchLine`, `RenderRoleFile`, `renderFile`, `ResumeContext`, `NewWatcher`).
  The struct-field changes (`VerbText`, `Watcher`, `orchCLI`, `Strand`, `StrandStatus`, `InboxIssue`, `intakeIssueView`, `Issue`) take none.
- **Granularity:** card 4 applies the same help-text rewrite to eight packages and reviews as one unit.
  Card 3 is `VerbText`'s own card, so the field does not need a separate card.
- **Test economy on cards 1, 2, 9, 14:** each new test or row pins a distinct surface.
  Card 2's `TestExitCode_HelpPaths` rows pin the mount in the real root, and its `cli_test.go` pins the command on a fixture root.
- **Verify:** `./...` in the root module covers `cmd/lyx`, `contracts/stencils`, `tools/sandbox` and every targeted package, and the order matches the plan template.
  Card 15 targets no Go package, and the only nested module (`internal/planglyph/testdata/glyphchain`) is untargeted.
- **ImpactSummary:** card 3's and card 9's state real blast-radius conclusions.
  `Board.SetDeps`'s callers are `internal/boardcli/cli.go:767` and the three boardengine test files, as card 9 says.
