# Card 6 — webster-config-invalid

**Edit:**
- `internal/webstercli#batcherLoadError`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#websterCLI.resolvePersistentPreRun`
- `internal/webstercli/verbs_test.go`
- `internal/websterengine#LoadConfig`
- `internal/websterengine/config_test.go`
- `internal/websterengine/doc.go`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/claims_test.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/configengine#ErrInvalid`
- `plan:internal/configengine#MarkInvalid`
- `plan:internal/batcher#ErrRetiredKey`

**Intent:** A webster verb whose config content is broken refuses with a flag Master stops on, and an unreadable config file reads as transient (D10's config half, D12's webster wording).
`LoadConfig` wraps its unmarshal, role-spec and numeric-knob validation failures with `configengine.MarkInvalid`, so together with card 1 every content failure of `webster.yaml` carries `configengine.ErrInvalid`.
The wiring's two config loads that a verb depends on, `batcher.Active` and `websterengine.LoadConfig`, route their errors through one refusal builder (`batcherLoadError` generalized to name the module's file):
- content failure (`errors.Is(err, configengine.ErrInvalid)`): the message names the file and ends `way forward: fix <file> under _lyx/config, then re-run the verb`, or, when the error wraps `batcher.ErrRetiredKey`, `way forward: run "lyx config reconcile --apply", then re-run the verb`;
- any other error (the file exists but cannot be read): the message names the file and ends `way forward: transient, re-run the verb`.
`resolvePersistentPreRun` prints a content failure with `output.ErrFields` carrying `config_invalid: true`, and an unreadable file with no `config_invalid`; both abort before the verb's body, so no state, report or plan changes, and a re-run after the fix proceeds as normal (record-batch then records the batch).
The pre-run writes the refusal's friction note through `noteEnvelope`, the path every verb refusal takes, since a pre-run refusal never reaches the `noteRefusals` tee.
Master's template gains a ladder rung: a refusal carrying `{"config_invalid": true}` from any verb ends the run with `outcome: stuck` and a `stuck_reason` quoting the refusal, without calling another verb.
Bound: the flag is set only for the parse-or-validation case, so an unreadable file or any non-config refusal never stops a run through that rung; the other modules' config loads in the wiring keep their plain refusals.
Grep for `batcher.yaml did not load` and `fix batcher.yaml under _lyx/config`; `TestPersistentPreRunE_BatcherSelection` in `verbs_test.go` gains steps asserting `"config_invalid":true` for a malformed profile and for a profile carrying `master_base` (with the reconcile way forward), its absence for an unreadable `batcher.yaml`, and `state.json` byte-identical after each.
`config_test.go` asserts `configengine.ErrInvalid` on a bad role spec and a bad numeric knob.
`claims_test.go` pins the new rung; `refusal-spec.md`'s webster section gains the config-invalid and config-unreadable rows; the package doc's Master-ladder text names the rung.

**ImpactSummary:** Changes the envelope and wording of webster's pre-run config refusals and adds one Master ladder rung; no verb body changes.
