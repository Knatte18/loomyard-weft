---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D6, D7, Acceptance criteria form-test entry)
    summary: the form test contract does not admit the board payload token, and a whitespace token check splits every D7 Use string
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D7, Open risks merge entry)
    summary: the D7 coupling test compares key sets only, so a wrong ? or | marker is never caught, and the merge token already marks the required upsert key optional
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D4)
    summary: the long-lived watcher renders the index from its own process tree, which goes stale after a deploy while the hook verb renders the new binary's tree into the same role file
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md (D2, D3)
    summary: runnable groups such as lyx config [module] with --print and --set fall outside the leaf-only audience and line rules, so their own verb never reaches the index
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D2, D3)
    summary: the runtime index walk sees cobra's completion subtree, whose leaves carry no audience, and no Decision says how an unannotated leaf renders
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D1)
    summary: replacing cobra's help command drops its shell-completion function and unknown-topic fallback unless re-implemented, and D1 states no bound for what the replacement may skip
  - id: F7
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md (D11)
    summary: the pointer filter's bound does not say whether a comment matches only when its whole body is the pointer form, or how the slug placeholder matches
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md (Open risks, lyx --json listing help)
    summary: whether lyx --json lists the new help module is an observable CLI choice the record can make now, not a plan-time risk
  - id: F9
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md (D2, D6, D9 rationale; Open risks registration entry)
    summary: measurement narrative and a perishable tally sit in decision-record.md, and the registration-route choice is stated twice
---

### [MEDIUM] The form test does not admit the board payload token

**Location:** `decision-record.md`, D6 bullets and its "Enforced by" paragraph, D7's `Use` list, and the Acceptance criteria entry beginning "The form test fails on".

**Issue:** D6 lists a board payload `{…}` token among "the forms above", so its test text admits it by reference.
The acceptance criterion names only `<arg>`, `[<arg>]` and `<arg>...` as accepted positionals, so a test written to the criterion fails on every board payload command.
The D7 tokens also contain spaces (`{slug|id, status}`, `upsert {slug, title?, …}`) and nested braces (`merge {…, set_status?: {slug|id, status}}`).
A test that splits `Use` on whitespace and checks each token turns `{slug|id,` and `status}` into two malformed tokens.
Neither Decision says how the test splits `Use` or which rule the payload token must meet inside its braces.

**Fix:** In D6, state that the form test reads a brace-balanced `{…}` span as one token, and checks only that the span is balanced.
Leave the keys inside it to D7's coupling test.
Add the payload token to the acceptance criterion's list of accepted positional forms.

### [MEDIUM] The coupling test cannot catch wrong optionality, and the merge token is already wrong

**Location:** `decision-record.md`, D7 (the `merge` line and the coupling-test sentence), and Open risks ("The merge payload's optionality…").

**Issue:** D7's test holds each `Use` key set equal to the parser's accepted key set.
That compares names only, so a `?` on a required key, or a `|` on two keys that can be sent together, passes it.
The Open risk says "the coupling test catches a mismatch" in optionality, which the test as defined cannot do.
The `merge` line already has this mismatch.
The parser in `internal/boardcli/cli.go` refuses a payload without `upsert` ("missing required field: upsert"), and its own `Long` says `upsert` is required, but D7 writes `upsert?: <task>`.
The board entry's proposal also writes `upsert:` without `?`.

**Fix:** Change the `merge` token to `{remove_slugs?, upsert: <task>, set_status?: {slug|id, status}}`.
Then either widen the coupling test to compare required keys, optional keys and exclusive pairs, or drop the claim from the Open risk and give optionality as review discipline.
The record names markers per key, so the wider test fits it best.

### [MEDIUM] The watcher's index goes stale after a deploy

**Location:** `decision-record.md`, D4 ("compute that string … from `cmd.Root()` of the process they run in").

**Issue:** Three of the five render sites live in `orchengine.Watcher`, which runs in the long-lived `lyx orch watch` process (`internal/orchcli/watch.go`).
The process builds the watcher once.
Neither `update-plugins.sh` nor the watcher restarts it after a binary change.
`CLAUDE.md` has the hub orchestrator run `update-plugins.sh` itself after every landing, so a new binary appearing under a live watcher is routine.
After that, the watcher's clear and compaction renders write the old binary's index.
The `resume-context` hook spawns a fresh `lyx` and writes the new binary's index into the same role file.
The orch calls the new binary, so the Goal's "always matches the running binary" fails for the binary the orch runs.
The index the orch sees also flips between the two trees with each cycle.

**Fix:** Name the binary the index must describe, which is the `lyx` the orch's pane resolves.
Bound the watcher's copy to it, either by computing the index through that binary at each render or by restarting or refusing a watcher whose build stamp differs.
If the record instead accepts the staleness until the next `lyx orch start`, say so in D4 and in Open risks.

### [MEDIUM] Runnable groups fall outside the leaf-only rules

**Location:** `decision-record.md`, D2 (group line form) and D3 ("Every leaf command (one with no subcommands)", "A group carries none").

**Issue:** `lyx config` has subcommands, so it counts as a group.
It is also an operator verb in its own right: `Use: config [module]`, with `--print` and `--set key=value` editing a module's config directly (`lyx config --help`).
Under D2 its line is `` `config`: <Short> ``, with no `[<module>]` and no flags.
Under D3 it carries no audience.
So the commonest config call never appears in the index, and the index's claim to be complete is false for it.
The audience test also does not cover it, so a future runnable group can skip the audience decision.

**Fix:** In D3, define a leaf by behaviour, not by having no children.
For example: a command with positional arguments or local flags of its own carries an audience and renders a leaf-style signature on its group line.
Add a matching row to the audience-test and index acceptance criteria.

### [LOW] The runtime index sees cobra's completion subtree

**Location:** `decision-record.md`, D2 and D3; Notes for the plan writer (walker skip).

**Issue:** `lyx help index` walks `cmd.Root()` at run time, where cobra has already added `completion` and its shell leaves (`lyx completion --help` answers today).
D3's audience test walks `newRoot()` before `Execute` and skips `completion` anyway, so those leaves never get an annotation.
The Notes cover the walker skip for tests only.
No Decision says whether the index skips `completion`, or what it prints for any leaf without an annotation.

**Fix:** In D2, state that the index skips cobra's `completion` subtree.
State that a leaf with no annotation is never printed, so the index shows only commands the tree-walk test has classified.

### [LOW] D1 gives no bound for what the replacement help command can skip

**Location:** `decision-record.md`, D1 ("so `lyx help [command]` keeps its cobra behaviour").

**Issue:** Cobra builds its default help command inside `InitDefaultHelpCmd` as an unexported literal.
It carries a `ValidArgsFunction` that completes `lyx help <tab>` with command names.
It also has an "Unknown help topic" fallback that prints the root usage.
A command installed through `SetHelpCommand` keeps neither unless the implementer copies them.
Both are guards that D1 says nothing about.
The only acceptance criterion is `lyx help board`, which reaches neither.

**Fix:** In D1, state what the replacement keeps: argument completion, the unknown-topic fallback (or a JSON error per `PATTERN-cli-cobra`), and routing to `index`.
Add an acceptance row for an unknown topic.

### [LOW] D11's pointer-filter bound leaves the match rule open

**Location:** `decision-record.md`, D11 (pointer comment bullet).

**Issue:** "Recognised by its one form" with "only that form is dropped" does not say whether the comment body must equal the form exactly, with any slug in the backticks, or only contain it.
A contains-match also drops a human comment that quotes the pointer sentence inside a longer text.
That lets through more than the stated bound.

**Fix:** State that a comment is dropped only when its whole body, trimmed, equals the pointer form with any slug.
Add a row to the import acceptance criterion: a human comment that contains the pointer sentence inside other text is kept.

### [LOW] Whether `lyx --json` lists `help` is a decision, not a risk

**Location:** `decision-record.md`, Open risks ("The `--json` help renderer skips a child named `help`…").

**Issue:** `renderCmdJSON` in `internal/clihelp/jsonhelp.go` skips any child named `help`.
Whether the new module shows up in `lyx --json` is observable CLI behaviour, and the record has every fact it needs to choose now.
Deferring it leaves the plan writer to set a CLI surface the record exists to fix.

**Fix:** Decide it in D1.
The recommended choice is to list `help` in `lyx --json` and keep skipping `completion`, since `help` is now a real module.
Then drop the Open risk.

### [NIT] Measurement narrative, a tally and one double statement in the decision record

**Location:** `decision-record.md`, D2 rationale, D6's third bullet, D9 rationale and Decision, and Open risks (the `SetHelpCommand` entry).

**Issue:** The D2 rationale (3.5k tokens, 11 to 12 percent) and the D9 rationale (ten largest pages, 40k tokens) are deliberation narrative that the plan writer cannot act on.
The D9 Decision's "the pages the operator measured at 2.5 to 5.4 KB" is the same narrative.
D6's "the 13 commands that spell them today" is a perishable tally; a grep for `[--` in `Use:` lines finds fewer today, since some spell required flags.
The requirement is that every command spelling an optional flag in `Use` is rewritten.
The registration-route choice appears in both Constraints and Open risks.
None of these carries a requirement that is not already stated elsewhere: the 2,000-byte cap and the rewrite obligation stay where they are.

**Fix:** Move the measurements into `support-log.md`'s Interview turn 7 or Rejected alternatives, where most already appear.
Reword D6's bullet as "every command that spells an optional flag in `Use` is rewritten".
Keep the registration-route choice in Constraints only.

## Verdicts on the focus entries without a finding

- D4 premise: holds.
  `startAutoReload` and `reloadAfterCompaction` in `internal/orchengine/watcher.go` call `startReload` with `skipsSkills` true and render the role file first.
  Only the clear cycle (`tickClearing`) loads skills.
  The render sites D4 names agree with the Notes: one in `orchcli/start.go` covers both start and `--adopt`, one is in `resumecontext.go`, and the three watcher calls make up "the watcher's reload render".
- `--audience internal` exposure: bounded.
  It prints only names and `Short`s that `--help` already shows.
- `set-deps` gaining `id`: bounded by D7's bound sentence, and `resolveLookup` already refuses a payload with both `slug` and `id`.
- D3 "a hidden command is `internal`" against Scope's "hiding is out": not a defect.
  The rule goes one way only (hidden implies `internal`, not `internal` implies hidden), so no command gets hidden.
- D11 fold heading depth against its acceptance criterion: consistent.
  `foldIssue` writes `## From issue #N`, and both texts put the comments one level below it.
- Registration-route Open risk: a real open choice between two plan-level mechanisms; only its double statement is flagged (F9).
