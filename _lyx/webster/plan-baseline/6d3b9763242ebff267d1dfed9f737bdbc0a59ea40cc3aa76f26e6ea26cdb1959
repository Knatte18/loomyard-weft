# Card 22 — rebaseline-card-index

**Edit:**
- `internal/websterengine#State`
- `internal/websterengine#Run`
- `internal/websterengine#Rebaseline`
- `internal/websterengine#fingerprintMismatchWayForward`
- `internal/websterengine/fingerprint.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/rebaseline_test.go`
- `internal/websterengine/fingerprint_test.go`
- `internal/websterengine/beginbatch_test.go`
- `internal/websterengine/runlevel_test.go`
- `internal/webstercli#websterCLI.rebaselineCmd`
- `internal/webstercli#persistPlanFingerprintRebaseline`
- `internal/webstercli/cli_test.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/planparser#OverviewWithoutCardIndex`

**Intent:** A card added after the last begun batch, with its Card Index line, lands through rebaseline, and the refusals for a done batch's card or any other overview change name that follow-up-card landing (D7), with the decision sequence documented (D8).
- The record: `State` gains `PlanOverviewFrameHash` (JSON `planOverviewFrameHash,omitempty`), the hex SHA-256 of `planparser.OverviewWithoutCardIndex` over `00-overview.md`; a helper in `fingerprint.go` computes it, and it is recorded wherever `PlanFileHashes` is: `restampBaseline` (in `fingerprint.go`), `Run`'s new-state branches, and `webstercli`'s `persistPlanFingerprintRebaseline`, which copies the restamped `PlanFileHashes` onto a reloaded state after a bracket verb rewrote the plan and now copies the frame hash beside it, so a failed bracket verb never drops it.
- Acceptance: when `00-overview.md` is among the changed plan files, `Rebaseline` accepts it if the state carries the frame hash and the file's frame hash now equals it, so the change is confined to the `## Card Index` section; the index change is then held to the existing card-set rule, which compares `NN-slug` ids, so only cards after the last begun batch can be added, removed or reordered, and a begun card's index line keeps its number and slug.
- Refusals: a state without the frame hash refuses any overview change as today; a changed frame (frontmatter, title, framing, `## verify:`, `## Shared Decisions`, `## Rename mechanic`) refuses naming the overview outside its Card Index; an edit to a card of a terminal `done` batch keeps refusing.
  Each of these refusals, and `fingerprintMismatchWayForward`'s changed-overview branch for begin-batch and run, names the landing that works: restore the card or the overview, or add a follow-up card after the last begun batch that carries the decision, with its Card Index line, then `lyx webster rebaseline --card NN` naming it.
  An index-only overview change in `fingerprintMismatchWayForward` names rebaseline with the added cards' `--card` flags instead of calling the overview never rebaselined.
Bound: outside the Card Index nothing may change, and a done batch's commits are never reopened by a plan edit; what passes is a reworded one-line intent of a begun card in a later Master render, while that card's file stays pinned by its recorded hash.
The package doc's rebaseline section states the acceptance and the mid-Webster operator-decision sequence: `lyx loom decision add` for the record, the card edit (or a follow-up card with its index line), then `lyx webster rebaseline --card NN`; no refusal or prompt text names a loom verb.
`rebaseline_test.go` gains rows: an added card after the last begun batch with its index line accepted, each framing-section change refused with the follow-up way forward, a begun card's renamed index slug refused, a done batch's card refused with the follow-up way forward, and a state without the frame hash refusing any overview change; `TestFingerprint` and `TestRestamp` in `fingerprint_test.go` gain rows pinning the frame hash and its restamp; `beginbatch_test.go` covers both overview branches of the mismatch way forward; `runlevel_test.go` asserts first init records the hash; every case is a row or step in an existing test, so no budget row changes.
In `webstercli/cli_test.go`, `TestPersistPlanFingerprintRebaseline` asserts the frame hash is carried onto the reloaded state beside `PlanFileHashes`, and `TestRebaselineCmd_Refusals`'s "named card of a done batch" row asserts the follow-up-card way forward in place of the reset route card 17 left it asserting.
The rebaseline long help is rewritten to match, and in `refusal-spec.md`, starting from card 17's text: `rebaseline: begun card of a done batch` and `rebaseline overview changed` name the follow-up-card landing in place of the reset route, `rebaseline overview changed` narrows its trigger to a change outside the Card Index, `plan fingerprint mismatch` rewrites its `00-overview.md` clause to the index-only rebaseline and the follow-up-card landing, and `plan changed before the verb's own rewrite`, which quotes that way forward, follows it.

**ImpactSummary:** Adds one persisted State field and lets rebaseline accept an index-only overview change; every other overview change and done-card edit stays refused with a new way forward.
