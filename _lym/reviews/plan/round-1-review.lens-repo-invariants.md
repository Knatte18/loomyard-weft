```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked reflective seam tests, tier tags, docs the card makes untrue, and card order. Two findings, both LOW.

**Per-card list**
- **Card 1**
  - Seam: held. `Task` gains `Discussed`, but no test in `internal/boardengine` reflects over `Task`'s fields; the `reflect.DeepEqual` hits are value checks.
  - Tier: held. Both new tests are tier 0.
  - Docs: broken, see F1. `boardengine/doc.go` is edited, but `internal/boardcli/doc.go:29` is not listed.
  - Order: held. It is the first card and needs nothing later; `boardcli/cli.go:460` reads `UpsertPayloadKeys`, and card 1 also edits the help in `boardcli#Command`.
- **Card 2**: seam n/a; tier held (tier 0); docs held (`doc.go`, the `Rename` doc comment and the `rename.go` header are all named); order held (uses only card 1).
- **Card 3**: seam n/a; tier held (tier 0); docs held; order held (uses card 1's `decisionFilePrefix`).
- **Card 4**: seam held (`replayFields` is not reflected over); tier held (rows added to tier-0 tables); docs held (`doc.go` and the `BoardDir` doc comment); order held (uses card 1 only).
- **Card 5**: seam n/a; tier held (a step in tier-1 `TestCLI`); docs held (`boardcli/doc.go` and `SKILL.md`); order held (uses card 1 only).
- **Card 6**: seam n/a; tier held (tier 0); docs held (`loomrecipe/doc.go` under Extends); order held (standalone).
- **Card 7**
  - Seam: held. `Env` gains `ApproveDiscussion`. The only non-test `shedrecipe.Env{` builders are `loomcli/wiring.go:628`, `battencli/wire.go:917`, `envkit.go:295` and a partial `Env` in `landingdeps.go:428`, which only feeds `CodePerchGate`. Card 7 covers the first three, and `TestWire_EverySeamFilled` (`//lym:seam internal/shedrecipe#Env`) is in its Uses.
  - Tier: held. Tiers 0 and 1, one each.
  - Docs: broken, see F2. These are comments inside files card 7 already edits.
  - Order: held. It uses card 6's `discussionApprovalRow` and card 1.
- **Card 8**: seam held (no reflective test over `decisionDeps`; the `reflect` hits in `decision_test.go` are value checks); tier held (tier 0); docs held (decision.go header and `loomcli/doc.go`); order held (uses cards 1, 6 and 7).
- **Card 9**: seam held (as card 8); tier held (tier 0); docs held (the group `Long`, the file header's "one verb" line, `loomcli/doc.go`); order held (uses card 8's fields, which come earlier).
- **Cross-card order**: held. Cards 2 and 4 need nothing from cards 3–9, and card 9 edits `decisionFake` after card 8 does.

**Findings**

F1
- Severity: LOW. Class: consistency.
- Location: card 1, `01-board-decision-record.md`.
- Summary: card 1 extends the "accepted only as the stored value" payload rule to `discussed`. `internal/boardcli/doc.go:29` documents that rule for `front` and `on_land` only. No card updates that line; card 5 edits `boardcli/doc.go` for `get` and `--record` only.
- Evidence: `internal/boardcli/doc.go:29`, "`upsert` takes `front` and `on_land` only as the entry's stored values, since only lym writes them."

F2
- Severity: LOW. Class: consistency.
- Location: card 7, `07-discussion-approve-seam.md`.
- Summary: two comments in files card 7 edits become untrue, and its Intent rewords neither.
  - The Plan-Bouncer comment in `tapestry-recipe.yaml` says "Neither Discussion-Bouncer nor Code-Bouncer gets this key: neither of their artifacts carries an approval flag". Card 7 rewords only the "only row in the whole list" clause.
  - `TestApproveSeam_FailsToBuild`'s doc comment says "approve_seam names env.ApprovePlan and nothing else, so any value other than "plan" is a build-time error". Card 7 changes only that test's row to `webster`.
- Evidence: `contracts/recipes/tapestry-recipe.yaml:300-305`; `internal/loomrecipe/approveseam_test.go:95`.

**Tool calls**
1. Grepped `//lym:seam` and `reflect.` in the touched packages, to find reflective seam tests the manifest lacked.
2. Read the tail of `replay_test.go`, grepped `NumField`/`TypeOf`, and read the `TestWire_EverySeamFilled` markers, to see whether any test reflects over `Task`, `replayFields` or `Env`.
3. Grepped the non-test `shedrecipe.Env{` builders and the per-recipe wire seam tests, to check that card 7 covers every `Env` builder.
4. Grepped docs, specs and `pattern/` for `approve_seam`, `ApprovePlan` and "only verb" text, to find docs falsified with no card listing them.
5. Grepped the `on_land` and `UpsertPayloadKeys` mentions in boardcli docs and tests and in `SKILL.md`, plus test placement, to find payload-rule docs card 1 leaves stale.
6. Grepped other `UpsertPayloadKeys` consumers, reflection in `decision_test.go`, `loomcli/doc.go` seam text and the orch role's `decision add` mentions, to check `decisionDeps` seam coverage and verb docs.
