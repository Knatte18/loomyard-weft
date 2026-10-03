---
format: 5
approved: false
root: internal
language: go
---

# Plan: test-suite-change-amplification

Make a small production change cost edits in few test files: shared seam fakes and builders move into `internal/testkit/<kit>/` under a new Testkit Invariant, git plumbing moves into `gitkit` under one spawn rule, `hubforge` becomes the only fabric fixture vocabulary, and the duplicate fixtures, decoders and assertions #324 and #325 list collapse onto one owner each.
Production behaviour is unchanged except for three exported fabricengine sentinels, whose error text stays byte-identical.
Cards follow D12's order: the testkit root first, then `gitkit` and the #325 items, then the kits and their package-by-package conversions, so the suite is green after every card.

## Card Index

1 — testkit-root — create the doc-only testkit package, its enforcing scan and the Testkit Invariant
2 — gitkit-query-helpers — add the git query and commit helpers to gitkit
3 — gitkit-spawn-rule — one shared gitkit spawn rule for the Tier Purity and Hermetic Env scans
4 — fabricengine-git-queries — fabricengine query helpers onto gitkit
5 — fabricengine-git-commits — fabricengine commit helpers and inline add/commit pairs onto gitkit
6 — git-helpers-other-packages — gitrepo, fabriccli, cmd/lyx, hubforge, ideengine, loomcli, landingshed git helpers onto gitkit
7 — webster-git-helpers — webster pair git scratch helpers onto gitkit
8 — hubforge-pair-and-open — add AddPairWith and OpenFabric to hubforge
9 — fabricengine-hub-fixture — delete fabricFixture, use *hubforge.Hub and h.Topology
10 — fabricengine-pair-and-open-sites — route repeated Add/Open setup through hubforge
11 — fabricengine-sentinels — export the refusal sentinels and assert with errors.Is
12 — fabriccli-dedupe — one owner each for remedy, help-prose and conflict-envelope assertions
13 — fabricengine-add-dedupe — one Add wiring test instead of replayed slug classes
14 — reedengine-faketmux — one scripted fakeTmux, replacing the bespoke recorders
15 — reedengine-faketmux-rest — convert the remaining execHook closures
16 — reedcli-tables — reap-cycle table and duplicate watchdog-spawn tests
17 — gitoracle-package — move the gitrepo parity oracle to its own leaf package
18 — locationkit — one shared synthetic lyxcwd.Location builder
19 — shuttlefake-kit — shared Engine and ReedOps fakes
20 — shuttlefake-webster — webster pair onto shuttlefake
21 — shuttlefake-cli — shuttlecli and loomcli smoke engines onto shuttlefake
22 — shuttleengine-fixture — one Runner/Run fixture in shuttleengine
23 — shuttleengine-run-literals — Run and Config literals through the fixture
24 — shedfake-kit — shared Shuttle, BurlerRunner, MergeShuttle, webster placeholders and Call helpers
25 — shedadapters-shedfake — shedadapters onto shedfake
26 — shedadapters-bouncer-fixture — one bouncer fixture in shedadapters
27 — landingshed-shedfake — landingshed onto shedfake with one local Deps builder
28 — producers-shedfake — loomshed, preflightshed, battenshed and loomcli Call sites onto shedfake
29 — shedrecipe-shedfake — shedrecipe onto shedfake with one local Env builder
30 — envkit-kit — shared full Env, landing Deps and rework Deps builders
31 — shedbuild-envkit — shedbuild onto envkit
32 — loomrecipe-envkit — loomrecipe onto envkit
33 — battenrecipe-envkit — battenrecipe onto envkit
34 — loomcli-wiring-reflect — one reflective every-seam-filled test in loomcli
35 — battencli-wiring-reflect — one reflective every-seam-filled test in battencli
36 — logcapture-kit — one shared logger capture
37 — envelope-kit — one shared JSON envelope decoder
38 — envelope-assertions — fabric, reed and cmd/lyx envelope assertions through the kit
39 — lyxbin-kit — one shared lyx binary builder
40 — slices-cleanup — containsPath and equalStringSlices replaced by slices
41 — old-policy-citations — drop the last old-policy citations and close the test-count accounting

## Shared Decisions

### Decision: test-count-accounting

- **Decision:** the leaf test count is the number of `(package, test)` pairs from `go test -json -tags integration ./...` that have no subtest of their own, computed with:
  `go test -json -tags integration ./... 2>/dev/null | jq -r 'select(.Action=="run") | .Package+"\t"+.Test' | sort -u | awk -F'\t' '{all[$0]; t=$2; while ((i=match(t, /\/[^\/]*$/))>0) { t=substr(t,1,i-1); parent[$1"\t"t] } } END { c=0; for (k in all) if (!(k in parent)) c++; print c }'`
  Card 1 records the count before its change, card 41 records the count after, and every card that deletes or consolidates a leaf test names each removed or renamed leaf in its commit body.
- **Rationale:** the decision record requires every drop to be itemised against an intended deletion; commit bodies keep that record in git, card by card.
- **Applies to:** cards 1, 12, 13, 16, 34, 35, 41, and any card whose conversion changes a test's leaf shape.

### Decision: kit-conventions

- **Decision:** a kit exposes exported fields and optional func overrides rather than option structs, fakes reproduce the richest copy they replace, the only assertions a kit makes are `t.Fatalf` on its own setup plus the `shedfake` `Call`/`RequireOutcome` and `envelope` `RequireOK`/`RequireErr` pairs, and a kit's non-test files avoid bare fabric vocabulary (the Fabric Vocabulary Invariant scans them as production files outside its owner set).
- **Rationale:** D1, D3 and the Fabric Vocabulary constraint in the decision record.
- **Applies to:** cards 18, 19, 24, 30, 36, 37, 39

### Decision: locationkit

- **Decision:** the shared `*lyxcwd.Location` builder #325 §6 asks for lives in `internal/testkit/locationkit`, a kit D2 does not name.
- **Rationale:** its three copies span `cmd/lyx` and `fabricengine`, so D1 puts it under `internal/testkit`; none of D2's kits fits a location builder, and `envkit` would import `fabricengine` through `landingshed`, a cycle for fabricengine's in-package test.
- **Applies to:** card 18

### Decision: cycle-forced-local-copies

- **Decision:** beyond D1's named cases (`shuttleengine`'s own fakes, `shedrecipe`'s and `landingshed`'s own builders), `internal/burlerengine` keeps its in-package `Shuttle` fake, because `shedfake` imports `burlerengine` for `BurlerRunner`'s types.
- **Rationale:** D1's rule that a package an import cycle bars from the kit keeps exactly one local copy.
- **Applies to:** cards 24, 29, 27, 22

### Decision: protected-tests-and-plants

- **Decision:** D9 governs every card: protected tests (invariant scans, finding-citing regressions, live `smoke`/`integration` tests, the gitrepo oracle, the `livestate_*` harness, `internal/loomcli/wiring_commitstatus_integration_test.go`) are restructured at most, every assertion survives, and a touched scan keeps its allowlist, floor and token set except where D3/D4 change them by name.
  Every plant proof is made, observed failing and removed inside its card; no plant is committed.
- **Rationale:** the suite does not have too many checks; this task moves where they live.
- **Applies to:** all cards

### Decision: test-hygiene

- **Decision:** in every test a card touches, doc comments restating the test name are dropped, no comment counts rows or names seams by list, and touched allowlists use one entry per line without aligned values (D10).
- **Rationale:** comment churn is a fifth of edited test lines.
- **Applies to:** all cards

## verify:

go build ./...
go vet -tags smoke ./...
go test ./...
go test -tags integration ./...
