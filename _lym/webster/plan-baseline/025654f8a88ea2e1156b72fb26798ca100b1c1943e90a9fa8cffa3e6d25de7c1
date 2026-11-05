# Card 13 — beam-pr-open-update

**Create:**
- `plan:internal/beamcli#prCmd` -> `func prCmd() *cobra.Command`
- `plan:internal/beamcli#beamPullRequestSide` -> `type beamPullRequestSide struct`
- `plan:internal/beamcli#prOpen` -> `func prOpen(ctx context.Context, deps landDeps, slug string) (map[string]any, error)`
- `plan:internal/beamcli#prUpdate` -> `func prUpdate(ctx context.Context, deps landDeps, slug string) (map[string]any, error)`
- `plan:internal/beamcli#judgePullRequestVerb` -> `func judgePullRequestVerb(deps landDeps, verb, slug string) error`
- `plan:internal/beamcli#TestPROpen` -> `func TestPROpen(t *testing.T)`
- `plan:internal/beamcli#TestPRUpdate` -> `func TestPRUpdate(t *testing.T)`
- `plan:internal/landingshed#OriginMissing` -> `func OriginMissing(origin Origin) string`

**Edit:**
- `internal/beamcli#Command`
- `internal/beamcli#landDeps`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#newLandWorld`
- `internal/beamcli#judgeLandOwners` -> `func judgeLandOwners(deps landDeps, owners boardengine.Owners, here, verb, slug string) error`
- `internal/beamcli#land`
- `internal/beamcli/doc.go`
- `internal/landingshed#TestResolvePullRequest`
- `PATTERN.md`

**Extends:**
- `internal/landingshed/doc.go`
- `internal/githubclient/doc.go`

**Uses:**
- `internal/landingshed#checkOrigin`
- `plan:internal/beamcli#judgeAwaitingRun`
- `plan:internal/beamcli#judgeNotReady`
- `internal/beamcli#judgeLandEntry`
- `plan:internal/pawlengine#ReadLanding`
- `plan:internal/pawlengine#UpdateLanding`
- `internal/pullrequest#API`
- `internal/pullrequest#Open`
- `plan:internal/testkit/prkit#API`

**Intent:** Add `lym beam pr open <slug>` and `lym beam pr update <slug>` (D6), thin verbs over `internal/pullrequest` the beam's orch calls, so the orch never runs `gh` by hand.

- `judgePullRequestVerb` runs, in order, `judgeLandEntry` (pair here, open beam), `judgeLandOwners` (another machine owns the beam or its parent, naming its adopt), `judgeAwaitingRun` and `judgeNotReady`. `judgeLandOwners` takes the verb, so its refusals name the verb in hand; `land` passes `lym beam land`.
- `beamPullRequestSide` holds `API pullrequest.API`, `Origin landingshed.Origin`, `Branch` (the beam pair's branch) and `ParentBranch`. `landDeps` gains `PullRequestSide func(slug string) (beamPullRequestSide, error)`, `PushBeam func(slug string) (head string, err error)`, `BeamHasCommits func(slug string) (bool, error)`, `Landing func(slug string) (pawlengine.LandingRecord, bool, error)`, `WriteLanding func(slug string, mutate func(rec *pawlengine.LandingRecord) error) error` (update the record, then commit the landing directory, a commit over an unchanged directory a no-op) and `Description func(slug string) (title, body string, err error)`. `hubLandDeps` fills them over the beam's pair: the origin facts as loomcli's `Origin` seam reads them (origin URL, the parent's upstream, the hub's push skip), the API through `pullrequest.Open`, so the token comes only through `internal/githubclient`; `BeamHasCommits` as whether the beam pair's HEAD holds a commit the local parent branch lacks, the read Finalize's `TaskHasCommits` makes of a task pair, records-only commits counted, so `lym beam pr open`, `lym beam pr close`, `lym beam ready` and Finalize judge an empty beam alike; the push through `fabricengine.PushPairAnchored`, under the remote-refs lock (`PATTERN-remote-refs-lock`); the record at `loomengine.BeamLandingPath`; the description through `summaryparser.Parse` of the beam's `summary.md`.
- `landingshed.OriginMissing` returns `checkOrigin`'s missing fact, "" when none, so the verb refuses an origin the resolver's origin check refuses (no GitHub origin, an unparsable one, no parent upstream, push skip set), naming the fact and the way forward: give the hub a GitHub origin, an upstream for the parent branch and no push skip, then run the verb again, or land without a pull request with `lym beam ready <slug>`.
- `prOpen`: refuses a landing record with no `DescribedAt` for this beam (naming `lym beam describe <slug>`), a beam with no commits beyond its parent (naming landing a member into the beam first, or landing without a pull request with `lym beam ready <slug>`), and a missing origin fact. A recorded pull request GitHub reports open answers `already: true`, opens nothing and commits the landing directory through `WriteLanding` with no change, so a record a failed commit left uncommitted is committed; one reported closed is reopened and answered `reopened`; one reported merged changes nothing and names `lym beam ready <slug>`, after which Finalize lands it. With none recorded it pushes the beam branch, finds an open pull request from it into the parent or creates one with the description's title and body, marks a found draft ready for review (`SetDraft` false) and leaves a found one's text as it is, then records `Number`, `URL`, and the pushed head as both `PushedHead` and `AnnounceHead`. A record write after a created pull request that fails is refused naming the pull request's URL and `lym beam pr open <slug>` again, which then finds it open and records it.
- `prUpdate`: refuses with no pull request recorded, naming `lym beam pr open <slug>`; pushes the beam branch, sets the title and body from the description with `Edit`, marks a draft ready for review, and records the pushed head as `PushedHead`.
- A failed push or GitHub call in either verb is refused naming its cause and the way forward: fix it, then run the same verb again.
- `prCmd` is the `pr` group under `beam`, with `open` and `update` here and `status` and `close` added by card 14; each help text says the orch runs it when the operator says so, that it acts only on the pull request from the beam's branch into its parent, that the orch never merges one, and its refusals. `Command` adds the group.
- Bound: the verbs open, edit and reopen only the pull request from the beam's own branch into its parent, read a token only through `internal/githubclient`, never merge, and change nothing once the ready record stands.
- `PATTERN.md`'s entry becomes, verbatim:

```
- `PATTERN-github-auth` — Calling GitHub: auth only via internal/githubclient; no other production package runs gh; only lym selfreport create files an issue; only PR-Review, Finalize, `lym beam pr` and pair teardown open, reopen, merge or close a pull request; `lym beam pr` never merges. (test)
```

Tests (tier 0): `TestPROpen` over `newLandWorld` with a `prkit.API`, its passing rows first clearing the fixture's default ready flag: none recorded, none open: pushed, created with the description's text, recorded with both heads; none recorded, an open draft found: marked ready, not edited, recorded; recorded open: `already`, no GitHub call but `Status`, `WriteLanding` called; recorded closed: reopened; recorded merged: nothing changed, `lym beam ready` named; a failing `WriteLanding` after a created pull request: refused naming its URL and `lym beam pr open`; a failing push: refused naming the rerun; refusals for no description stamped, no commits, each missing origin fact, a standing ready record, a run not at Beam-Await, a beam another machine owns, each naming its way forward. `TestPRUpdate`: pushed, edited, draft marked ready, head recorded; no record refused naming `lym beam pr open`. `TestResolvePullRequest` gains `OriginMissing` rows matching each origin refusal.

**ImpactSummary:** a new `beam pr` verb group in beamcli over pullrequest and the landing record; `judgeLandOwners` gains a verb parameter with its one caller `land`; landingshed exports one origin helper.

**Commit:** 13: Add lym beam pr open and update
