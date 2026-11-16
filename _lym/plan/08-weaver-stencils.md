# Card 8 — weaver-stencils

**Create:**
- `contracts/stencils/weaver/weaver-template-batch.md`
- `contracts/stencils/weaver/weaver-body-verify-fix.md`
- `plan:contracts/stencils#WeaverTemplateBatch` -> `var WeaverTemplateBatch []byte`
- `plan:contracts/stencils#WeaverBodyVerifyFix` -> `var WeaverBodyVerifyFix []byte`
- `plan:internal/weaverengine#BatchPromptInputs` -> `type BatchPromptInputs struct`
- `plan:internal/weaverengine#RenderBatchPrompt` -> `func RenderBatchPrompt(in BatchPromptInputs) ([]byte, error)`
- `plan:internal/weaverengine#TestRenderBatchPrompt` -> `func TestRenderBatchPrompt(t *testing.T)`

**Extends:**
- `contracts/stencils#entries`
- `internal/weaverengine/doc.go`

**Uses:**
- `contracts/stencils/webster/webster-prefix-recovery.md`
- `contracts/stencils/webster/webster-body-verify-fix.md`
- `contracts/stencils#TestRegistry_MatchesOnDiskTree`
- `contracts/stencils#TestParentDirectiveScans_SyntheticRows`
- `internal/runplan#RenderCardGates`
- `internal/runplan#RenderBatchGate`

**Intent:** Weaver's two prompts, read at call time through `stencilstore` (D4, D8, D10; PATTERN-stencil-ownership).

- `weaver-template-batch.md` is the batch attempt's whole prompt, cold like Webster's recovery prefix, with no master session behind it.
  Its markers: `parent_directive`, `edit_directive`, `pattern_directive`, `friction_directive`;
  `cards`, the batch's card pointers with their card gates, and `batch_gate`;
  `decision_record`; `journal`, which the agent reads first; `report_path`;
  `failure_digest` and `uncommitted_paths`, each `none` on a first attempt and a clean tree;
  `gate_tiers` and `rename_tools`.
  It tells the agent to commit its work on the task branch and to write its report last, with `status`, `head_sha`, optional `need` and `renames`, and `journal`: what the batch struggled with and what later batches must keep in mind.
  It never names a `lym webster` or `lym weaver` verb.
- `weaver-body-verify-fix.md` is the verify fixer's body, joined behind `gatefixer`'s cold frame: it reads the findings and the journal, fixes the failing identities, commits `fix: <summary>` and writes its reply.
- `contracts/stencils`' registry gains both, as `WeaverTemplateBatch` and `WeaverBodyVerifyFix`, so the stencils seed and `TestRegistry_MatchesOnDiskTree` sees them; the parent and edit directive scans cover the batch template.
- `RenderBatchPrompt` reads the template through `stencilstore.Read` and fills every marker from `BatchPromptInputs`:
  the cards and the gates `runplan.RenderCardGates` and `runplan.RenderBatchGate` render (the last batch with the main packages), the decision record and journal paths, the report path, the failure digest and uncommitted paths of a retry, and the directives through `parentdirective`, `editdirective`, `pattern.Directive` and `friction.Directive`.
  It writes the prompt under the prompts directory and returns its bytes.

`TestRenderBatchPrompt` is tier0: on a first attempt the values Go fills name the batch's card files and gates, the decision record, the journal and the report path, and the digest and paths read `none`;
on a retry they carry the given digest and uncommitted paths;
a batch that is the partition's last renders the batch gate with the main packages, and one that is not renders it without them.

**ImpactSummary:** The stencil registry gains two entries and the weaverengine doc grows; no existing entry changes.
