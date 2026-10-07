# Card 6 — named profiles in batcher.yaml

**Edit:**
- `internal/batcher/config.go`
- `internal/batcher/config_test.go`
- `internal/batcher/registry.go`
- `internal/batcher/identity.go`
- `internal/batcher/template.yaml`
- `internal/batcher/doc.go`
- `internal/loomshed/batchifier.go`
- `internal/loomshed/gates_test.go`
- `internal/configreg/configreg.go`
- `internal/configreg/configreg_test.go`
- `internal/webstercli/wiring.go`
- `internal/webstercli/verbs_test.go`
- `internal/webstercli/cli_test.go`
- `internal/websterengine/runlevel_test.go`
- `contracts/specs/refusal-spec.md`
- `PATTERN.md`
- `docs/overview.md`

**Delete:**
- `internal/batcher/batcher_test.go`
- `internal/batcher/registry_test.go`

**Uses:**
- `plan:internal/batcher#NewCost`
- `plan:internal/batcher#CostParams`
- `plan:internal/batcher#Identity`

**Intent:**
`batcher.yaml` names profiles and `active:` picks one (Decision 8).
The template ships an `identity` and a cautious `cost` profile with `active` left on identity, so no run groups cards after this card.

**Schema.**
`active:` names a profile; `profiles:` maps a name to `batchifier` (`identity` or `cost`) and, for `cost`, `alone_above`, `budget`, `max_cards` and the `weights` map card 2 parses.

**Open map** (`config.go`, `configreg.go`).
`profiles:` is keyed by names the operator chooses, so `config.go` gains `ConfigOpenMaps()` returning `profiles`, and the shared load helper passes it to `configengine.LoadOrTemplate`.
`configreg.Modules()` sets it on the `batcher` entry, so `lyx config reconcile` carries the operator's profiles whole instead of pruning them, and the template's profiles fill in only when the file holds no `profiles:` key.

**Registry** (`registry.go`).
A map literal from batchifier kind to a constructor taking the profile name and profile, returning a `Batcher` or an error; the `cost` kind builds `NewCost` with its `CostParams`.
It replaces `register`, `lookup`, `Select` and identity's `init()` registration; `DefaultName` stays the name an empty `active:` resolves to.

**Resolution** (`Active`, still through `configengine.LoadOrTemplate`).
- An empty `active:` and `active: identity` resolve to the identity batchifier even when no `profiles:` block exists, so every existing `batcher.yaml` keeps working; a configured profile named `identity` wins over that default.
- An `active:` naming a configured profile builds it through the registry, the returned `Batcher` named after the profile.
- Load errors, each naming `batcher.yaml` and the offending profile or key: `active:` names no profile; an unknown `batchifier` kind; for a `cost` profile, a missing parameter, `alone_above` or `budget` not positive, `max_cards` below 2, and card 2's missing, negative or unknown coefficient, which reaches `Active`'s caller unchanged.

**Template** (`template.yaml`, comments in the existing style).
`active: ""` stays the first line so the existing `strings.Replace` in `verbs_test.go` keeps working.
`profiles:` holds `identity: {batchifier: identity}` and `cautious`: `batchifier: cost`, `max_cards: 3`, `alone_above: 1200000`, `budget: 2000000`, and weights `startup_context: 40000`, `fork_messages: 4`, `target_messages: 6`, `test_file_messages: 1`, `uses_messages: 2`, `context_per_line: 12`, `package_context: 2000`.
All values are in context tokens; card 8 rescales them from the calibration report.

**Refusals.**
- The webstercli prologues (`wiring.go`, both hub and standalone) wrap an `Active` error as `webster: batcher.yaml did not load: <cause>; way forward: fix batcher.yaml under _lyx/config, then re-run the verb`.
  `PersistentPreRunE` runs before any verb's body, so the refusal lands before a verb touches state; a rebaseline whose profile does not load leaves the recorded partition and state unchanged by the same route.
- `loomshed`'s `batchifierWayForward` becomes `; way forward: fix batcher.yaml (its active: key or the profile it names), then re-step`; the Batchifier and Webster rows already map a load error to Stuck with it.
- `refusal-spec.md`: the loom `batcher misconfigured` row's trigger becomes "`batcher.yaml` does not load: `active:` names no profile, or a profile has an unknown kind or a missing or invalid parameter", way forward as above; the webster section gains a `batcher.yaml did not load` correctness-halt row with the webstercli way forward.

**Tests.**
`config_test.go`'s `TestActive` table covers: the template resolving identity; empty and `identity` `active:` with no `profiles:` block resolving identity; `active: cautious` on the template resolving a cost batcher named `cautious`; and one error row per load error `Active` adds (no such profile, unknown kind, a missing or non-positive threshold, `max_cards` below 2), each asserting the error names `batcher.yaml`;
the no-such-profile row is a file whose `profiles:` holds only an operator profile and whose `active:` names `cautious`, which fails to load only while the template's profiles are not filled into a present `profiles:` key, so it pins the open map too; the missing, negative and unknown coefficient cases stay in card 2's `ProfileWeights` rows, and the `verbs_test.go` step below pins that a coefficient error reaches `Active`'s caller naming `batcher.yaml`.
`configreg_test.go`'s `TestLookup` gains a `batcher` row asserting `OpenMaps` is `profiles`.
The old `TestSelect` and the register/lookup round trip go with the code they test (`batcher_test.go`, `registry_test.go` deleted).
`verbs_test.go`'s `TestPersistentPreRunE_BatcherSelection`: the unknown-profile step asserts the new message; a new step seeds a `state.json`, runs `rebaseline` under a `cost` profile with a negative coefficient, and asserts exit 1, a message naming `batcher.yaml`, and `state.json` byte-identical.
`gates_test.go` asserts the new way-forward text.
The `batcher.Select` calls in `cli_test.go`, `verbs_test.go` and `runlevel_test.go` become `batcher.Identity()`.

**Docs.**
`doc.go` describes profiles, kinds and resolution.
`PATTERN.md`'s `PATTERN-batcher-registry`: webster's execution unit is the batch the active profile's batchifier derives, the batchifier kind selected through `internal/batcher`'s registry and the profile by `batcher.yaml`'s `active:` key.
`docs/overview.md`'s batcher bullet: profiles, identity the default, the cost batchifier available.

**ImpactSummary:** `batcher.Select` and the self-registration go, their callers are tests only; every existing `batcher.yaml` resolves as before; `batcher.yaml` load errors now name the file in webster and in the loom rows; `configreg` gains one `OpenMaps` value for `batcher`, which only stops `lyx config reconcile` from pruning operator profiles.
