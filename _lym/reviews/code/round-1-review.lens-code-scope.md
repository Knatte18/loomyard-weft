```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace each change in the diff to the board entry's goal points, and each goal point to the changes that build it.

**Decisions.** The record's `## Goal` board entry has two goal points. The record has no `Added after Discussion` entries.
- **G1**: let `branch_prefix` be set per machine or per user, with the hub default as the fallback.
- **G2**: review which other keys are personal rather than hub-wide (PR identity, Slack operator, default model or effort, `batten.open_ide`) and give them the same layer.

**Changes, one per purpose, and what each traces to.**
- **C1**: `internal/fabricengine/config.go` moves `branch_prefix` from `CategoryCrossPair` to `CategoryMachine`. It also rewrites the `ConfigSpec` doc and the `Config` doc ("every pair shares" becomes "fabric's settings"). Traces to G1, which it builds through the machine layer, a valid choice of "per machine or per user". The doc rewrites are forced by the category move.
- **C2**: the widened machine-category definition in `internal/configengine/layers.go` (the `CategoryMachine` comment), `internal/configengine/doc.go` and `pattern/PATTERN-config-layers.md`. Forced by C1: the rule's text would otherwise not cover a person's convention.
- **C3**: the `branch_prefix` comment in `internal/fabricengine/template.yaml`, the `internal/fabricengine/doc.go:767-768` paragraph and the `docs/overview.md:519-520` examples. Forced by C1, which made them stale.
- **C4**: `internal/configcli/configcli_test.go` repoints the "no machine key" row from fabric to slack. Forced by C1, since fabric now has a machine key.
- **C5**: `internal/configreg/configreg_test.go` adds `wantMachineKeys` for fabric. Forced by C1.
- **C6**: the new subtest in `internal/fabricengine/config_test.go`, where the machine override wins over the hub file. Forced by C1; it is the test of G1.

**Decisions and the changes that build them.**
- G1: C1, with C2–C6 forced by it.
- G2: none. The diff makes no change for G2. The review result lives only in `_lym/landing/summary.md` under "The other personal keys stay where they are".

No change in the diff traces to no decision.

**Findings**

1. **MEDIUM, class `scope`.** Location: the diff as a whole, against the board entry's G2.
   - Summary: G2 is not built. The entry asks to review the other personal keys and give them the same layer. The diff gives none of them a per-machine layer. `batten.open_ide`, `seed_ide`, `open_window`, `landing.pr_reviewer` and `agents.efforts` stay `CategoryNone`, so no machine-wide value is possible.
   - How it was decided: the writer declined on their own call, with no `Added after Discussion` entry recording it, and the summary does not name it as a parent correction.
   - Evidence for the summary's category claims, which are accurate: `internal/battenshed/config.go:65-67`, `internal/landingshed/config.go:137`, `internal/agentconfig/config.go:124`, `internal/shuttleengine/codexengine/config.go:77`.
   - Why it matters: the decision record's `## Goal`, under "## Idea" (second bullet) and "## When", names this review as part of the task. The only reason given for not building it is in the summary's "Design decisions": giving those keys the machine layer would need a change to `configengine`'s layer rule, "kept out of this change".
   - Fix: either build the per-machine layer for those keys, or have the parent record the narrowing with `lym loom decision add`. Leave the code as it is until the parent decides.

**Not filed**
- The entry's "When" says the work is planned after the beam `adopt-build` lands, and the summary says it has not landed. Starting the run before then is the dispatcher's call, not something the diff can change.

**Tool calls**
1. A grep of the `Categories` entries for `open_ide`, `seed_ide`, `open_window`, `pr_reviewer` and `efforts`, plus a list of the `slackengine` files that mention the operator. The manifest did not cover the categories of the other personal keys, which G2 asks to review.
