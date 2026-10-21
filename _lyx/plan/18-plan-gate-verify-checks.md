# Card 18 — plan-gate-verify-checks

**Create:**
- `internal/planparser#VerifyCommands`

**Edit:**
- `internal/loomshed#PlanContext`
- `internal/loomshed#planContextFindings`
- `internal/loomshed#TestNewPlanGate`
- `internal/loomcli#TestValidatePlanCmd`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#loomCLI.wire`
- `internal/shedrecipe#planContextOf`
- `internal/shedrecipe#Env`
- `internal/testkit/envkit#FullEnv`
- `internal/planparser#checkVerifyNestedModule`
- `internal/planparser#checkVerifyPackageMissing`
- `internal/planparser#checkVerifyModuleWide`
- `internal/planparser_test#TestValidate_VerifyNestedModule`
- `internal/planparser_test#TestValidate_VerifyPackageMissing`
- `internal/planparser_test#TestValidate_VerifyModuleWide`
- `internal/planparser/validate.go`
- `internal/planparser/doc.go`
- `contracts/specs/loom-plan-spec.md`

**Uses:**
- `internal/loomshed#seedPlanContextFixture`

**Intent:**
The plan gate stops judging the retired `## verify:` section and starts judging card `**Verify:**` lines against the hub's `refuse_raw` list (D9, D10).

`PlanContext` gains `RefuseRaw []string`, the hub's `gate.yaml` list; `loomcli`'s validate-plan wiring loads it through `gateslot.LoadConfig` beside `language.yaml`, and `shedrecipe`'s plan-gate entry reads it from a new `Env.RefuseRaw` the wiring fills, `envkit` filling it with `["go test"]`.
At both loads, `validatePlanCmd`'s and the wiring's that fills `Env.RefuseRaw`, an absent file (`errors.Is(err, gateslot.ErrConfigAbsent)`) leaves `RefuseRaw` empty, as card 17's `Claude.Prepare` does, so `lyx loom validate-plan` stays usable in a repo with no `gate.yaml`; every other load error refuses the verb as the language load does.
`planContextFindings` adds, in order after its existing checks:
- `verify-raw-command`, blocking: a card's `**Verify:**` line holding a `refuse_raw` prefix in command position, read over `planparser.VerifyCommands`, which exports the chain split the existing verify checks use (`&&`, `||`, `;` and `|`; a `**Verify:**` value is the one line after its label), and past a leading `env`, `exec`, `xargs`, `sudo` or `time` launcher, naming `lyx gate test`, because Go re-runs that line outside any agent hook;
- `verify-gate-no-wait`, blocking: a `lyx gate test` call in a `**Verify:**` line lacking `--wait`, naming `--wait`;
- `plan-verify-retired`, informational: an overview carrying a `## verify:` section, saying no gate runs it; the parser keeps accepting the section, so an in-flight plan still parses.
The two `**Verify:**` checks read only cards that are not done, since a done card is frozen and `done-card-edited` refuses the rewrite the check would ask for: `ValidatePlan` passes `planContextFindings` only its pending cards, and `ValidateReworkPlan` passes the fresh generation's cards, numbered from `first_card`, which holds no done card.
The reaching input for each is the card or overview text itself, after every format check passes; `TestNewPlanGate`'s context-check table reaches each.
Bound: like the shell guard, the check reads static text, so a test hidden behind a script in a `**Verify:**` line passes it.

The plan-level checks on the retired section go: `checkVerifyPackageMissing` and `checkVerifyNestedModule` stop reading the overview chain and keep judging card lines; `checkVerifyModuleWide`'s way forward names `lyx gate test --wait --tier <n> --card <id>` in place of the `--tags` form.
`validate.go`'s check list, `doc.go` and `loom-plan-spec.md`'s validation-checks section record the three new IDs, the dropped overview reads and the reworded way forward; the spec's `verify-raw-command` description names "a command the hub's `refuse_raw` lists" and never spells a listed prefix, so card 17's denied-recovery scan stays green.

Tests: `TestNewPlanGate`'s context-check table, which counts each context check ID over a `ValidatePlan` call, gains one want-count column per new ID and a severity check for the informational one, with `RefuseRaw` set on the context as `RenameTools` is: a `**Verify:**` line `go test ./internal/x`, one `cd x && go test ./...` and one `env go test ./x` refused with `refuse_raw: ["go test"]` and passed with an empty list, `lyx gate test ./internal/x` refused naming `--wait`, `lyx gate test --wait --tier 0 --card 3` passed, a done card's raw line not reported, and an overview with `## verify:` reported informational and not blocking.
The validate-plan wiring's `gate.yaml` load failure refuses the verb as the `language.yaml` failure does; `TestValidatePlanCmd` gains the row of an unparsable hub `gate.yaml` refusing the verb with the loader's way forward, and the row "no gate.yaml, the verb runs and `verify-raw-command` reports nothing".
No validate fixture needs a seeded `gate.yaml` then: loomcli's `hubLocationAt` and `glyphPlanFixture` keep loading as they do today.
`TestValidate_VerifyNestedModule` and `TestValidate_VerifyPackageMissing` swap their overview-chain rows for rows showing the overview is no longer read; `TestValidate_VerifyModuleWide` asserts the new way forward.

**ImpactSummary:** the plan gate gains two blocking and one informational context check and stops reading the overview's verify chain; two context constructors carry the new list.
