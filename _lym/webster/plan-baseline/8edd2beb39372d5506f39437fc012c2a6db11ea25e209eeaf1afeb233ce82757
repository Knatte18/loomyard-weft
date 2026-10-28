# Card 4 — The envelope and status carry restep_hold for the LLM driver

**Create:**
- `internal/shedverbs#Spec.restepHolds`
- `internal/loomcli#TestSpecFor_RestepHold`

**Edit:**
- `internal/shedverbs#Spec`
- `internal/shedverbs#loopObject`
- `internal/shedverbs#loopObject.fields`
- `internal/shedverbs#statusCmd`
- `internal/shedverbs#runLoop`
- `internal/shedverbs/doc.go`
- `internal/loomcli#loomCLI.specFor`
- `internal/shedverbs#TestStatusCmd_FoundEnvelope_GenericCore`
- `internal/shedverbs#TestLoop_RunsUntilStopAndPrintsOneEnvelope`
- `internal/shedverbs#assertLoopKeys`
- `internal/shedcli#loopRun.assertLoop`
- `internal/loomcli#TestStatusCmd_EnvelopeKeySet`
- `contracts/stencils/shed/shed-template-driver.md`
- `contracts/stencils/shed/shed-template-driver-guide.md`

**Extends:**
- `contracts/specs/loom-status-spec.md`

**Uses:**
- `internal/shedverbs#Spec.handsBack`
- `internal/landingshed#EscalateHalt`
- `internal/landingshed#EscalateStopPrefix`

**Intent:**
D3's LLM-driver half: the LLM driver branches only on envelope and status fields, so `lym shed status` (and `lym loom status`, built on it) and the loop's envelope gain a recipe-neutral `restep_hold` key beside `hand_back`, read the same way.

`Spec` gains `RestepHold func(producer, reason string) bool`, the recipe's told hold check; `Spec.restepHolds(st)` is true when `RestepHold` is told, `st.State` is blocked and `RestepHold(st.CurrentProducer, st.Error)` answers true, and false otherwise, including when no check is told.
`statusCmd`'s found envelope adds `"restep_hold": spec.restepHolds(st)` to its core map; `loopObject` gains `RestepHold`, `runLoop` sets it from `spec.restepHolds(after)` beside `HandBack`, and `loopObject.fields` renders it as `restep_hold`, so the loop object's closed key set grows by that one key.
`loomCLI.specFor` tells `RestepHold: landingshed.EscalateHalt` when `c.recipe` is `shedrun.RecipeLoom` or `shedrun.RecipeDarn`, so loom and darn tell the escalate predicate, and pawl and a receiver with no recipe tell none.
The predicate stays the prefix's only reader: the envelope carries its answer, never the text.

The driver stencil never re-launches on `last_step.binary_changed` and starts no binary watch at a stop marked `restep_hold`, as at one marked `hand_back`, though such a stop still goes to the arbiter first; only the arbiter's `resume` or the orch's `lym loom resume` moves it.
Bound: the key only withholds the LLM driver's own build re-launch, so it can let nothing through; a stop it holds still has the arbiter and the orch as its way forward.

Tests:
- `TestStatusCmd_FoundEnvelope_GenericCore` asserts `restep_hold` false with no check told in its generic rows, and gains restep-hold rows built like its hand-back rows: a blocked Finalize status under a check that answers true for it reads true, the same status with no check told and a failed Finalize status under the check read false.
- `TestLoop_RunsUntilStopAndPrintsOneEnvelope` tells a `RestepHold` check answering true for producer B beside its `HandBack` check, gains `wantRestepHold`, true for the blocked row only, and asserts `loop["restep_hold"]`.
- `assertLoopKeys` and `loopRun.assertLoop` add `restep_hold` to the closed key set, the latter's count going from 12 to 13; `TestStatusCmd_EnvelopeKeySet` adds `restep_hold` to its wanted keys.
- `TestSpecFor_RestepHold` (tier0, table-driven over a hand-populated receiver per recipe) pins that `specFor`'s `RestepHold` answers true for Finalize with an `EscalateStopPrefix` reason and false for a plain reason on loom and darn, and that pawl and an empty recipe tell none.

Docs: `internal/shedverbs/doc.go` lists `restep_hold` among the loop object's keys and says `lym shed status` reports it at its top level, beside `hand_back`.
The loom status spec gains a `restep_hold` section after its `hand_back` section.
The driver stencil's re-launch list, its never-re-launch line and its binary-watch step, and the guide's binary watch, skip a stop marked `restep_hold` as they skip one marked `hand_back`, while such a stop still goes to the arbiter first.

**ImpactSummary:** one new told `Spec` field and one envelope key in `internal/shedverbs`, told only by loom's `specFor`; the closed loop-key-set assertions in `internal/shedverbs`, `internal/shedcli` and `internal/loomcli` grow by the key, and nothing reads it in Go.
