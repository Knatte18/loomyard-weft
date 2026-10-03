---
format: 5
approved: true
language: go
---

# Plan: test-suite-derived-guards

Part 2 of the test-suite cleanup (#326, #327 and the board's socket-leak item), building on part 1's `internal/testkit`.
Every invariant scan moves onto one harness, `internal/testkit/scankit`;
the CLI-tree, config-registry, path-constructor and recipe-row tests derive their expectations from the code they guard;
plan and stencil fixtures go through `plankit` and `stencilkit`;
raw stencil wording is pinned in one claims table;
shedrecipe conformance, the `loomcli`/`shedverbs` duplicates and the commit-status seam are each tested once;
and `tmuxkit` isolates every live-tier test package's tmux sockets, enforced by a new scan.
The order follows D24: harness and Testkit restatement first, then the scan conversions package by package, the derived registry tests, #327's items, and `tmuxkit` with its invariant last.
Each card leaves the suite green.

## Card Index

1 — scankit — add the stdlib-only source-scan harness kit with its unit tests
2 — testkit-scan-on-scankit — rebuild the Testkit scan on scankit and restate the Testkit Invariant
3 — cmdlyx-git-guards — move the cmd/lyx git and fabric guards onto scankit, dropping go env GOMOD
4 — cmdlyx-module-guards — move the remaining cmd/lyx scans and root lookups onto scankit
5 — tier-purity-hermetic-env — move Tier Purity and Hermetic Env onto scankit and retire stale entries
6 — lyxcwd-scans — move lyxcwd's enforcement, docs-link and raddle scans onto scankit
7 — leaf-guards — turn every leaf import guard into a short scankit caller
8 — seam-guards-a — move the first half of the seam guards onto scankit
9 — seam-guards-b — move the second half of the seam guards onto scankit
10 — package-local-scans — move the remaining package-local invariant scans onto scankit
11 — cli-tree-walk — replace the hand-kept help tree with one newRoot walk
12 — per-cli-copies — delete the per-CLI NoArgs, UnknownSubcommand, Short and exact-verb-set copies
13 — configreg-properties — assert the config registry by property plus a registration scan
14 — path-rule-table — merge the path guards into one rule-driven constructor table
15 — recipe-row-table — pin the loom recipe row list once with a perch-shape property
16 — export-plan-format — export planparser's plan-format version constant
17 — plankit — add the test-side valid-plan writer and restate the Planparser invariant
18 — plankit-webster — convert the webster-side plan fixtures to plankit
19 — plankit-loom — convert the loom-side plan and glyph-repo fixtures to plankit
20 — stencilkit — add the registry-driven stencil seeding kit
21 — stencilkit-webster-orch — convert webster, burler, treadle and orch stencil seeders
22 — stencilkit-loom — convert the loom, shedadapters, friction and pattern stencil seeders
23 — claims-table — create the stencil claims table with contracts/stencils' own wording checks
24 — claims-webster — move websterengine's raw wording checks into the claims table
25 — claims-engines — move loomengine, burlerengine and shedadapters wording checks into the claims table
26 — shedrecipe-conformance — test unknown-key rejection once over the whole registry
27 — shedverbs-duplicates — delete loomcli's duplicate shedverbs tests after moving unique cases
28 — statuscommit-seam — extract the shared commit-status seam into one package
29 — wide-call-helpers — route wide-function test calls through one local helper per file
30 — tmuxkit — add the tmux isolation kit with its Testkit and Tier Purity rules
31 — tmuxkit-testmain-a — run the first half of the tagged-test packages through tmuxkit.Main
32 — tmuxkit-testmain-b — run the second half of the tagged-test packages through tmuxkit.Main
33 — tmuxkit-socket-fixture — convert hand-rolled tmux keys to tmuxkit.Socket
34 — tmux-isolation-scan — enforce the Tmux Test Isolation Invariant with a cmd/lyx scan

## Shared Decisions

### Decision: scankit-api

- **Decision:** `scankit` exports `Root(t)` (module root via `runtime.Caller` and `go.mod`), `Walk(t, Options, fn)` returning the scanned-file count, where `Options` names roots (default the module root), an optional absolute base for fixture trees, a file filter (`Prod`, `Test`, `All`), extensions (default `.go`) and a shallow package-directory mode, and `fn` receives a `File` with `Rel` (slash-separated), `Abs`, `Data` and a lazy `AST(mode)`;
  `Allowlist`, built from `[]Entry{{Key, Why}}`, with exact and path-prefix matching, recording what matched, and `Stale()`/`RequireNoStale(t)`;
  `RequireFloor(t, scanned, min, what)`;
  `AssertImportAllowlist(t, pkgDir, allowed...)`.
- **Rationale:** D1 leaves the API to the plan; fixing the names lets the conversion cards agree.
  Card 1 may adjust a signature if a conversion needs it, as long as the later cards use the result consistently.
- **Applies to:** cards 1–10, 13, 30, 34

### Decision: skip-dir-set

- **Decision:** `scankit`'s shared skip set is the union of the two sets in use today: `.git`, `_lyx`, `.lyx`, `_mill`, `.scratch`, `.wiki`, `_raddle`, and any directory whose name contains `testdata`.
  A converted scan whose scanned set changes because of the union and whose verdict or floor is affected is itemised in its commit body.
- **Applies to:** cards 1–10

### Decision: scan-conversion-protocol

- **Decision:** A converted scan keeps its matching logic, allowlist entries, floor and token set verbatim, apart from D2's two tightenings: stale-entry detection on its allowlist, and a floor (at least one scanned file) where it had none.
  A stale entry the tightening finds is removed in the same card.
  Each converted scan is proved by planting one violation, watching it fail, and removing the plant.
  The card's commit body lists each proof and each stale entry removed, so the PR description can itemise them.
- **Applies to:** cards 2–10, 13, 30, 34

### Decision: allowlist-form

- **Decision:** Every allowlist a touched scan carries is written one entry per line without gofmt-aligned values: a `[]string`, or a slice of `{entry, reason}` pairs (D8).
- **Applies to:** every card that touches an allowlist

### Decision: test-accounting

- **Decision:** A card that deletes or moves a test names each one in its commit body, with where its assertion now lives when it moved.
  Leaf test counts (top-level tests without subtests plus every subtest, from `go test -json`) are recorded before card 1 and after the last card, and every drop is itemised against a deletion the decision record names.
- **Applies to:** cards 11, 12, 15, 23–28

### Decision: protected-tests

- **Decision:** Per D21, every `CONSTRAINTS.md` invariant scan, every regression test whose comment cites a finding, crucible round, sabotage proof or live observation, every `smoke`/`integration` live test, the gitrepo oracle, the fabricengine `livestate_*` harness and `wiring_commitstatus_integration_test.go` are restructured at most and never deleted.
  A protected assertion leaves its test only for the new owner the decision record names (the CLI walk, the claims table, `shedverbs`' tests, `statuscommit`).
- **Applies to:** all cards

### Decision: test-comments

- **Decision:** A touched test drops doc comments that restate its name, and no comment counts rows or lists seams, modules or verbs by name (D22).
- **Applies to:** all cards

### Decision: tmuxkit-api

- **Decision:** `tmuxkit` exports `Main(m *testing.M) int`, the `TestMain` entry point a package calls as `os.Exit(tmuxkit.Main(m))` after its own setup, and `Socket(t, tmux string) string`, the per-test fixture key with `kill-server` in `t.Cleanup`.
  Every export except `Main` counts as a spawn for Test Tier Purity.
- **Applies to:** cards 30–34

### Decision: root-lookups-left-alone

- **Decision:** Tests that resolve the repo root through `runtime.Caller` but enforce no `CONSTRAINTS.md` invariant (`cmd/lyx/lydriveskill_test.go`, `internal/loomcli/driveskill_test.go`, `tools/deploy/main_test.go`) are not converted.
- **Rationale:** D2 binds invariant scans;
  none of these runs `go env GOMOD`, so D4 does not reach them either.
- **Applies to:** cards 3–10

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

go build ./...
go vet ./...
go test ./...
go test -tags integration ./...
go vet -tags smoke ./...
