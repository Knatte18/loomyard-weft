# Card 11 — finalize-beam-record

**Edit:**
- `internal/landingshed#ResolvePullRequest` -> `func ResolvePullRequest(slug, parentMode, entryField string, beam BeamRecord, parentRuleRequired bool, origin Origin, taskHasCommits bool) PullRequestDecision`
- `internal/landingshed#resolveWithParentRule` -> `func resolveWithParentRule(ctx context.Context, name string, deps Deps, parentMode, entryField string, beam BeamRecord, origin Origin, taskHasCommits, recordFound bool) (PullRequestDecision, error)`
- `internal/landingshed#PRReview.decide`
- `internal/landingshed#PullRequestDeps`
- `internal/landingshed#prLanding`
- `internal/landingshed#prMergeRetryWindow`
- `internal/landingshed#Finalize.pullRequestLanding`
- `internal/landingshed#Finalize.openAtOnce`
- `internal/landingshed#Finalize.mergeRefusalReason`
- `internal/landingshed#Finalize.landUnderLock`
- `internal/landingshed/doc.go`
- `internal/landingshed#TestResolvePullRequest`
- `internal/landingshed#TestFinalize_PullRequest`
- `internal/loomcli#landingDeps`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`
- `PATTERN.md`

**Create:**
- `plan:internal/landingshed#BeamRecord` -> `type BeamRecord int`
- `plan:internal/landingshed#NotABeam` -> `const NotABeam BeamRecord = iota`
- `plan:internal/landingshed#BeamWithoutPullRequest` -> `const BeamWithoutPullRequest BeamRecord = 1`
- `plan:internal/landingshed#BeamWithPullRequest` -> `const BeamWithPullRequest BeamRecord = 2`
- `plan:internal/landingshed#BeamPullRequest` -> `type BeamPullRequest struct`

**Uses:**
- `internal/pullrequest#MergeRetryWindow`
- `internal/pawlengine#ReadLanding`
- `internal/pawlengine#DropLandingPullRequest`
- `internal/loomengine#BeamLandingPath`
- `internal/loomengine#BeamLandingLockPath`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** Finalize lands a pawl run by the beam's landing record (D8) and keeps a beam pull request's text (D9); the resolver stays the one decision and takes the beam record as an input (`PATTERN-pull-request-mode`).

- `ResolvePullRequest` gains `beam BeamRecord`: `NotABeam` decides exactly as today. For a beam the entry's `pull_request` is not read (callers pass ""), and a pull request is wanted when the record holds one (`BeamWithPullRequest`, demand "the beam's landing record holds a pull request"), when beam.yaml is `required` or `merge`, or when GitHub's rules require one; `on` and `off` with none recorded want none. Every beam yes is `MergeAtOnce`, since a beam always lands by squash on GitHub with no review row. The origin checks apply as today. `resolveWithParentRule` passes `beam` through; `PRReview.decide` passes `NotABeam`.
- `PullRequestDeps` gains `BeamRecord func() (BeamPullRequest, bool, error)`, the beam landing record's pull request, and `DropBeamRecord func() error`, which drops it from the record and commits the landing directory; both nil on every recipe but pawl. `BeamPullRequest` holds `Number` and `URL`. `prMergeRetryWindow` is `pullrequest.MergeRetryWindow`.
- `Finalize.pullRequestLanding` with `BeamRecord` set never calls `EntryField` and reads the beam record under the parent lock beside the scratch record:
  - A pull request recorded: GitHub reports it open: `ResolvePullRequest(.., BeamWithPullRequest, ..)` says yes and the landing merges it on GitHub, squashed, with no edit of its title or body. Closed: halt naming the operator reopening it on GitHub and `lym loom resume`. Merged at the task head: the already-landed path, as today. Merged at another head: `CommitsMissing(merged, task head)` lists the beam commits GitHub's merge lacks; with none, the landing goes through the already-landed path, an empty squash after `syncParent` pulled the parent; with some, the landing is decided as with none recorded, below, so a `required`, `merge` or rules-required parent finds or opens a new pull request from the beam branch and merges it, and an `on` or `off` parent lands them by the local squash after `syncParent` pulled GitHub's merge in.
  - None in the beam record and an open pull request in the scratch record (one Finalize opened before a halt, or one the old binary's PR-Review left): merged on GitHub as one Finalize opened, with no text edit, whatever the resolver says.
  - None recorded and `ResolvePullRequest(.., BeamWithoutPullRequest, ..)` says yes (`required`, `merge` or GitHub's rules): marked `open` and `atOnce`, so `openAtOnce` finds an open pull request from the beam branch or opens one from the description, and keeps it in the scratch record, as today.
  - None recorded and no: the local squash landing and push, as today; a parent push refused as protected halts with today's way forward.
  - A failed `BeamRecord` read is an error stop.
- `openAtOnce` edits a found pull request's title and body only when `BeamRecord` is nil (D9); a pull request it creates gets the description's text. The squash commit's title and message still come from the description with the pull request's number and the credit block.
- `mergeRefusalReason`, for a beam run, names GitHub's message, `lym beam pr status <slug>` and `lym loom resume`, and never `lym batten cancel`, for a refusal still pending past the window, every `RefusalOther` and a `RefusalMethod`; a `RefusalMethod` keeps today's clause asking to allow squash merging in the repository's settings before the resume.
- `Finalize.landUnderLock` calls `DropBeamRecord` only once the parent-side landing completed, after `pushParent` answers Done, for a recorded pull request merged on GitHub and for one merged at another head; a failed drop is logged, since the landing has happened.
- `loomcli.landingDeps` fills both seams for a run whose recipe has a Beam-Await row, over `pawlengine.ReadLanding`/`DropLandingPullRequest` at `loomengine.BeamLandingPath` and a `fabricengine.CommitAnchoredPaths` of the landing directory; `intentionallyNil` in loomcli and battencli name both as beam-only, and envkit's `LandingDeps` fakes them.
- `PATTERN.md`'s entry becomes, verbatim:

```
- `PATTERN-pull-request-mode` — Deciding if a landing uses a pull request: only landingshed's one resolver, from the receiving `beam.pull_request`, the entry's `pull_request` or a beam's pull-request record, the parent's GitHub rules, origin and task commits; no run setting carries it; a yes merges on GitHub.
```

- So the overview keeps its headroom once this card and card 13 grow it, two entries drop wording only, as the decision record's added entries say, each line becoming, verbatim:

```
- `PATTERN-owner-machine` — Run start, resume, push or landing: only if no other machine owns its entry; a loom push reconciles run-scope origin first; owners only on roots and beams, set only by `boardengine`'s setter; others read their `hubgeom.ParentOf` owner; a dobby continue is no resume. — [background](pattern/PATTERN-owner-machine.md)
```

```
- `PATTERN-heddle-instructions` — Rendering heddle entries into a file or prompt: only via `heddleengine`, from its shipped layer, Bolt's `heddle.json` and, under `lym_override`, lym overrides; `heddle.json` written only via `lym heddle` or its replayer; fabric names no heddle file. — [background](pattern/PATTERN-heddle-instructions.md)
```

Tests (tier 0): `TestResolvePullRequest` gains beam rows: recorded yes under `off`; `on` and `off` with none recorded no; `required`, `merge` and a rules-required parent yes and `MergeAtOnce`; an entry field never read for a beam; `NotABeam` rows unchanged. `TestFinalize_PullRequest` gains beam rows: a recorded open pull request squash-merged with `Config.Squash` false, no `Edit` call, no `EntryField` call, record dropped after the parent push; a crash re-entry finding it merged at the task head lands through the already-landed path; merged at another head lacking nothing lands locally and drops; merged at another head lacking a beam commit under `required` opens a new pull request and merges it, and under `on` lands locally, each dropping the record; closed halts naming reopening and `lym loom resume`; pending past the window, `RefusalOther` and `RefusalMethod` halt naming `lym beam pr status` and `lym loom resume` and not `lym batten cancel`, the method halt also naming the squash setting; a failing `BeamRecord` read is an error stop; none recorded with an open scratch record merges it with no `Edit`; none recorded under `required` finds an open one without editing it, or opens one, and merges; none recorded under `on` lands locally with no GitHub call. A task run's rows keep passing, an `openAtOnce` of a found pull request still editing it.

**ImpactSummary:** the resolver and its internal wrapper gain a parameter, with every caller in landingshed and its tests listed; Finalize gains two optional seams filled only for pawl, and tapestry, darn and sampler land as today.

**Commit:** 11: Finalize lands a beam by its landing record and keeps its pull request's text
