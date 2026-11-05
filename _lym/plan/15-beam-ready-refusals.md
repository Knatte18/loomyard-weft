# Card 15 — beam-ready-refusals

**Edit:**
- `internal/beamcli#markReady`
- `internal/beamcli#readyCmd`
- `internal/beamcli#landDeps`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#newLandWorld`
- `internal/beamcli#TestMarkReady`
- `internal/beamcli#TestHubDobbyDeps_LandsOnlyAFlaggedBeam`
- `internal/beamcli/doc.go`
- `internal/beamcli#mergeParentCmd`

**Create:**
- `plan:internal/beamcli#judgeReadyLanding` -> `func judgeReadyLanding(ctx context.Context, deps landDeps, slug string) error`
- `plan:internal/beamcli#TestHubLandDeps_ReadyReads` -> `func TestHubLandDeps_ReadyReads(t *testing.T)`

**Uses:**
- `plan:internal/beamcli#judgeAwaitingRun`
- `plan:internal/beamcli#parentRuleRequired`
- `internal/beamcli#openMergeTask`
- `plan:internal/beamcli#beamPullRequestSide`
- `plan:internal/pullrequest#ReadMergeState`
- `internal/landingshed#ResolvePullRequest`
- `plan:internal/landingshed#BeamWithoutPullRequest`
- `plan:internal/landingshed#OriginMissing`
- `plan:internal/pawlengine#ReadLanding`
- `plan:internal/testkit/prkit#API`

**Intent:** `lym beam ready` becomes the point of no return and refuses, each with its way forward, every condition the orch could still fix at Beam-Await (D3).

- `markReady` keeps its signature, so no caller changes, and runs `judgeReadyLanding` under `context.Background()`: its one wait, `ReadMergeState`'s retry of an `unknown` state, is bounded by `pullrequest.MergeRetryWindow`; it keeps `judgeLandEntry` and still reports a beam already flagged as `already` without judging or rewriting (a record left uncommitted is still committed). For a beam not flagged it runs `judgeReadyLanding` before writing the record, in this order:
  1. `judgeAwaitingRun` (card 12): a run not at Beam-Await, running or halted, names `lym batten status <slug>`.
  2. `deps.Refusal(slug)`, the landing refusal `battencli.BeamLandRefusal`: open members and child pairs, each named as that refusal does.
  3. `ParentUnmerged(slug)`: true when the parent pair's local tip, or its upstream tip fetched under the remote-refs lock when the parent has an upstream and the hub's push skip is unset, is not an ancestor of the beam's head; refused naming `lym beam merge-parent <slug>`, whose merge-in fetches and merges the fresher of the parent's local and upstream tips, or `lym batten status <task>` of the open merge-in task `openMergeTask` finds. A parent with no upstream is judged by its local tip alone.
  4. The landing record names no `DescribedAt` for this beam, naming `lym beam describe <slug>`; or `DescriptionFindings(slug)` finds the description failing validation, naming each finding and `lym beam describe <slug>`.
  5. No pull request recorded: `PullRequestMode` gives the mode and the rules (card 14's `parentRuleRequired`, which reads no rule on a hub with no GitHub origin or no usable token), `BeamHasCommits(slug)` (card 13) gives `hasCommits`, and `ResolvePullRequest(slug, mode, "", BeamWithoutPullRequest, rulesRequired, origin, hasCommits)` decides. A `PullRequestMode` or `BeamHasCommits` error refuses naming the cause and `lym beam ready <slug>` again. A `Halt`, which Finalize would halt on, is refused with a message built from `landingshed.OriginMissing(origin)` (card 13), not the `Halt` text, whose way forward ends in `lym loom resume`, a step that does nothing for a run waiting at Beam-Await: it names the demand (the parent's beam.yaml pull_request or GitHub's rules), the missing origin fact and the way forward: give the hub a GitHub origin, an upstream for the parent branch and no push skip, then run `lym beam ready <slug>` again. Every `Halt` of `ResolvePullRequest` comes from a missing origin fact, so `OriginMissing` covers the same inputs. A yes for a `required` parent or GitHub's rules, read from the mode and rules themselves since every beam yes is `MergeAtOnce`, names `lym beam pr open <slug>`; a `merge` parent passes, since Finalize opens its own; a no passes, a beam with no commits beyond its parent included, since Finalize lands it with no pull request.
  6. A pull request recorded: `ReadMergeState` with the beam's head. A `PullRequestSide` error or a `ReadMergeState` error (no usable token, a changed origin, a failed call) refuses naming the cause and the way forward: fix it, then run `lym beam ready <slug>` again; `lym beam pr status <slug>` reads the same state. Open with a head other than the beam's names `lym beam pr update <slug>`; open and not mergeable now names each cause with its action, `dirty` and `behind` naming `lym beam merge-parent <slug>`, `blocked` the review or required check owed on GitHub, `draft` `lym beam pr update <slug>`, and an `unknown` still not computed naming a later `lym beam ready <slug>`, each with `lym beam pr status <slug>` to read it again; closed names `lym beam pr open <slug>`, which reopens it, or `lym beam pr close <slug>`. Merged passes when `MergedLacks(slug, mergedHead)` lists nothing: the merged head is the beam's head, or the beam's first-parent commits past it only merge the parent in; Finalize lands it (card 11). Merged lacking beam commits is refused naming those commits and the way forward `lym beam pr close <slug>`, then land without a pull request or open a new one with `lym beam pr open <slug>`.
- `landDeps` gains `ParentUnmerged func(slug string) (bool, error)`, `DescriptionFindings func(slug string) ([]string, error)` and `MergedLacks func(slug, mergedHead string) ([]string, error)`; `hubLandDeps` fills them over the beam's pair through fabricengine (the parent tips through the pair's `ReadParentTips` with fetch, which takes the remote-refs lock (`PATTERN-remote-refs-lock`), and the ancestry through the pair), `summaryparser.ValidateDescription`, and the beam's first-parent log past the merged head, a commit counting as a parent merge when its second parent is reachable from the parent branch.
- `mergeParentCmd`'s Long and `internal/beamcli/doc.go`'s merge-parent paragraph say the orch runs the merge-in before `lym beam ready`, which refuses while the parent holds commits the beam lacks, and that Finalize merges the parent in again at landing.
- `internal/beamcli/doc.go`'s ready paragraph is rewritten: the verb refuses, in order, a run not at Beam-Await, open work, an unmerged parent, a missing or invalid description and the pull request conditions; it is final, and no verb removes the record. Its sentence that the verb refuses what `land`'s entry check refuses and reports a beam already flagged stays, no longer as the whole contract.
- `readyCmd`'s help states that ready is final, that no verb removes the record (Finalize's landing, Beam-Down and the retired-row move end it), that the operator runs it with the orch only when satisfied, and lists the refusals; its way forward names the dobby landing the beam.
- `TestHubDobbyDeps_LandsOnlyAFlaggedBeam` writes the ready record through `MarkReady` directly instead of `markReady`, since its fixture's beam is not at a judged Beam-Await; `TestHubLandDeps_MarkReadyCommitsAnUncommittedFlaggedRecord` is unchanged, since its flagged fixture takes the `already` path.
- Bound: refusals 3 and 6 read a moment; after ready only the parent and GitHub can move, since card 7's guards start no member and the beam verbs refuse; Finalize merges the parent in again and halts, with a way forward, on a merge GitHub refuses.
- Reaching inputs: each refusal is reached by a beam at Beam-Await whose earlier checks pass and whose fake seam answers the failing fact.

Tests: `TestMarkReady` (tier 0) gains rows, each refused with its way forward and no record written: a run at Finalize; an open member; a parent tip not an ancestor (naming `lym beam merge-parent`), and the same with an open merge-in task (naming that task); no description stamped; a description finding; a required parent with none recorded; a rules-required `on` parent with none recorded; a required parent whose origin has no upstream, and a `merge` parent with the same missing fact (each naming the missing fact and `lym beam ready`, not `lym loom resume`); a failing `PullRequestMode` and a failing `BeamHasCommits` (each naming the cause and `lym beam ready` again); a recorded pull request whose `prkit` `Status` call fails (naming the cause and `lym beam ready`); a recorded head other than the beam's (naming `lym beam pr update`); `behind` (naming `lym beam merge-parent` and `lym beam pr status`); `blocked` (naming the owed check); closed (naming `lym beam pr open` and `close`); merged lacking a beam commit (naming it and `lym beam pr close`). Passing rows: all clear writes the record; merged at the beam's head passes; merged with only a parent merge past it passes; a `merge` parent with none recorded passes; a required parent, a beam with no commits beyond its parent and none recorded passes refusal 5 and writes the record; an already flagged beam answers `already` with no seam but `Ready` called. The new `TestHubLandDeps_ReadyReads` (tier 1) drives the three closures and card 14's `PullRequestMode` over a hub fixture: `PullRequestMode` for an `off` parent whose origin is the fixture's local path, not on GitHub, answers `off` with no rule and no error, reading no rules; `ParentUnmerged` false for a beam holding its parent's tip, true after a parent commit, and for a parent with no upstream judged by its local tip; `MergedLacks` empty for a merged head followed only by a merge of the parent and naming a beam commit made after it; `DescriptionFindings` naming a finding for a description with an empty body and none for a valid one.

**ImpactSummary:** `lym beam ready` gains six refusals over three new landDeps seams and the seams of cards 12 to 14; `markReady` keeps its signature; the dobby's landing and `lym beam land` are unchanged.

**Commit:** 15: lym beam ready refuses what the orch can still fix
