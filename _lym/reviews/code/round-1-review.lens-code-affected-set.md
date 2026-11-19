```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, I found what calls, reads or asserts it and checked whether the diff leaves any of those wrong.

**Per change, the affected set and whether it is covered**

1. **The operator-ban line now routes questions to the parent** (`contracts/stencils/parent/parent-directive-operator-ban.md:4`).
   - Its renderer and the variant it fills: covered. `parentdirective.Directive` puts the line under `parent-directive-parent.md`, right after "Ask it with `lym creel post --parent --expect-reply`", so "this way" points at that command. The no-parent variant (tackler, arbiter) does not render the line.
   - The tests that pin it: covered. `parentdirective_test.go:17`, `seatengine/prompt_test.go:114,117,327`, `burlerengine/prompt_test.go:312` and `loomengine/discussion_test.go:94` all key on "never the way forward", which the new line keeps.
   - The docs: covered. `parentdirective/doc.go:17` names the line only as the "operator-ban line", which still holds.
   - The interactive callers: covered. `samplerengine/spec.go:106` and `landingshed/prsession.go:112` pass `true`, and the drift judge uses `JudgeInteractive`. Those sessions keep asking in their pane, which `orch-role.md` "Questions from a run's agents" now states.
2. **`bouncer-fragment-decision.md:9`**: covered. Nothing pins the old wording; my grep found no test asserting the old or the new sentence. The new text agrees with the operator decision ("The parent answers itself or asks the operator").
3. **`burler-step-fix.md:44`**: covered. Nothing pins the old wording.
4. **Hold notifier on the gate card fixer** (`gatecli/cardfixer.go`, `newStrandRig`).
   - Both callers get it: covered. `newCardFixer` is at :186 and `newFixerStopper` at :223; the stopper never waits, so its notifier is never called and does no harm.
   - The routing: covered. `orchcli.NotifyAsker` and its doc (`orchcli/doc.go:337`) say the notice goes where `--parent` goes, and with an empty slug it signs as `&lt;shortname&gt;:GATE`. That matches the claim in `gatecli/doc.go`.
   - The shuttle docs: covered. `shuttleengine/doc.go:145-150` describes notifiers generically, so nothing there goes stale.
   - The orch-role docs: covered. Messaging (`orch-role.md:481`) describes the hold notice generically, and the new section points to it. The section exists at `orch-role.md:445`.
   - The program name: covered. `gateProgram` is a single named constant and appears nowhere else.
5. **New loom key `notify_parent_on`**.
   - `Config`, the `ConfigSpec` category (none, so it is open to the local layer and `run_settings`), and `template.yaml`: covered. `TestModules_EveryTemplateKeyClassified` is satisfied.
   - `docs/overview.md` and `orch-role.md`: covered.
   - `loomcli/doc.go`: covered.
   - Every other file that lists loom keys: covered. Grepping for the sibling `code_review_parent_timeout_min` turns up only `review.go` and the tests besides the files the diff already touches.
6. **`newShed` wraps the watched rows** (`loomcli/arm.go`).
   - Its callers: covered. `buildLoomShed` (`sharedbootstrap.go:480`) and `prReviewSendBackEdge` (`prfindings.go:301`) both call it. The second only reads `shed.Producers` for edges, and wrapping changes only `.Producer`, so the names and edges it reads stay intact.
   - The env poster: covered. `env.NotifyParent` (`wiring.go:994`) prefixes `loom &lt;slug&gt;: `, so the notice line does not open with `[row reached]`. `orch-role.md:285` correctly says the mail "holds" it, not "opens" it.
7. **New `shedrun.RowNoticeRecord`, `ReadRowNotice`, `WriteRowNotice` and `RowNotice`**.
   - `shedrun/doc.go`: covered.
   - PATTERN-lym-files: covered. The record is a cache under `.lym` in `ScratchDir`, and an untracked file may keep its lock as `path+".lock"`.
   - PATTERN-tolerant-records: covered. Reads and writes go through `internal/state`.
   - `TestPathConstructors` (`internal/shedrun/paths_test.go:33-105`): **not covered**. Its doc says it "pins every path constructor's layout", and it lists every scratch accessor, `WakeRecord` (`:96-97`) included, but the diff adds no `RowNoticeRecord` row. The reach report agrees: "no mapped test reaches internal/shedrun.RowNoticeRecord".
8. **Commit messages and departures**: there are no `Pattern-Departure:` or `Loosened-Check:` trailers in the range, and none is needed. No PATTERN changes are declared, so the "declared PATTERN changes landed" check has nothing to verify.

**Findings**

- **LOW | consistency** | `internal/shedrun/rownotice.go:24` (`RowNoticeRecord`)
  The new scratch path constructor has no row in `TestPathConstructors`, the table whose doc claims it pins every path constructor's layout.
  Evidence: `internal/shedrun/paths_test.go:33` (the doc) and `:96-97` (the sibling `WakeRecord` row); the reach report says no test reaches `RowNoticeRecord`.
  Fix: add a `{"RowNoticeRecord", …, filepath.Join(ScratchDir(l, "self"), "row-notice.json")}` row.

No other member of the affected set is left wrong.

**Tool calls**
1. Grepped for "never the way forward" and the old bouncer and burler wording, to find the tests and docs that pin the changed stencil text; the manifest had no list of them.
2. Grepped orch-role, overview, the orchcli doc and the shuttleengine doc for "hold notice" and the program signatures, to see which docs describe hold-notice senders; the manifest lacked these spans.
3. Read `orchcli/doc.go:325-345`, PATTERN-leaf-packages, cmd import guards, `orch-role.md` Messaging and the shuttleengine notifier doc, to check the docs of NotifyAsker and SetNotifier and any import rule on gatecli.
4. Listed the `orch-role.md` headings, grepped overview for notifier docs, and checked the shedrun path test, to confirm the "Messaging" section exists and to find the path-constructor test.
5. Read `shedrun/paths_test.go:30-110`, to check whether the constructor table claims to be exhaustive.
6. Grepped docs, orch-role, specs, the parentdirective doc and the gatefixer doc for descriptions of the operator-ban line, to find stale prose; the manifest had not covered these docs.
