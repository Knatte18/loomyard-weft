# Card 12 — housekeeping-inflight-prune

**Edit:**
- `internal/fabricengine#Topology`
- `internal/fabricengine#housekeepStores` -> `func housekeepStores(l *lyxcwd.Location, slug string, probe InFlightProbe)`
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine/housekeeping_integration_test.go`
- `internal/fabricengine/doc.go`
- `internal/pairteardown#Teardown.topology`
- `internal/reedengine/overlay_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/fabricengine#InFlightProbe` -> `type InFlightProbe func(slug string) ([]string, error)`
- `plan:internal/fabricengine#Topology.SetInFlightProbe` -> `func (t *Topology) SetInFlightProbe(probe InFlightProbe)`
- `plan:internal/fabricengine#gcPruneExpire` -> `const gcPruneExpire = "1.day.ago"`
- `plan:internal/pairteardown#Teardown.otherPairSessions` -> `func (t *Teardown) otherPairSessions(slug string) ([]string, error)`
- `plan:internal/pairteardown#sessionsOfOtherPairs` -> `func sessionsOfOtherPairs(live, pairSessions []string) []string`
- `plan:internal/reedengine#IsNoServer` -> `func IsNoServer(err error) bool`
- `internal/pairteardown/inflight_test.go`

**Prosa:**
- `//PATTERN.md`
- `//pattern/PATTERN-hub-store-gc.md`

**Uses:**
- `internal/reedengine#ListSessions`
- `internal/reedengine#ServerName`
- `internal/reedengine#SessionName`
- `internal/reedengine#LoadConfig`
- `internal/fabricengine#List`

**Intent:** D8 and D9 in `housekeeping.go`, the sole gc site (`PATTERN-hub-store-gc`).
`InFlightProbe` answers, for the pair being removed, the live reed sessions that belong to other pairs; `housekeepStores` takes it with the slug, writes the two keys as today, then skips the gc when the probe returns sessions, logged at Info with them, or an error, logged at Warn with it, and runs the gc only on an empty answer.
A nil probe answers empty: `Topology.Remove` has thirty-odd test callers and the pair-teardown chokepoint test pins its arity, so the probe rides on `Topology` through `SetInFlightProbe` rather than a re-sign, and `Remove` passes `t.inFlight` with the slug; the bound is that production reaches `Remove` only through `pairteardown`, which the chokepoint test enforces, and `Teardown.topology` sets the real probe there, so both callers of removal, `lyx fabric remove` and the batten teardown, carry it.
`Teardown.otherPairSessions` is the real probe: it loads the prime's reed config for the tmux binary, lists the hub socket's sessions through `ListSessions(cfg.Tmux, ServerName(prime.HubPath))`, maps a listing error `IsNoServer` admits to no sessions and returns any other error, lists the code repo's linked worktrees through `List`, drops the main entry and the removed slug's worktree, maps the rest through `SessionName`, and returns `sessionsOfOtherPairs(live, pairSessions)`, the pure intersection; the prime's own sessions, the orch among them, are never in the pair set and never count.
`IsNoServer` reports whether a tmux error wraps tmux's `no server running` stderr, so that wording stays in reed, the package that owns tmux's text.
`housekeepStores` also passes `-c gc.pruneExpire=1.day.ago` beside its existing settings, from `gcPruneExpire`, so unreachable loose objects older than a day are pruned and the loose count falls (D9); reflog expiry and `gc.worktreePruneExpire=never` are unchanged.
`doc.go`'s housekeeping paragraph, `PATTERN.md`'s `PATTERN-hub-store-gc` line and its background name the in-flight skip and the one-day expiry, replacing the sentence that gc deletes only objects past git's own default expiry; housekeeping remains no destructive primitive.
Tests: `TestStoreHousekeeping` gains subtests setting a fake probe through `SetInFlightProbe`: sessions returned write the keys and leave the sampled loose objects unpacked with the Info line logged, an error does the same with a Warn, an empty answer packs; and a D9 subtest writes unreachable loose objects with `git hash-object -w`, backdates one by two days with `os.Chtimes`, lowers the threshold and asserts the old one is gone and the young one kept after `Remove`.
The new untagged `internal/pairteardown/inflight_test.go` holds `TestSessionsOfOtherPairs` over live and pair session lists, so `internal/pairteardown` rises by one in the budget; `TestListSessions` in `reedengine` gains a row asserting `IsNoServer` on the no-server error and not on another failure.
The tmux listing itself runs only in production and the `tmux` tier, which this plan compiles and never runs.

**ImpactSummary:** `Remove`'s signature is unchanged for its thirty-two callers; the gc now skips while another pair's session is live and prunes day-old loose objects, both only on the removal path, and `Topology` gains one optional field.
