```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code the fix changed (lens fix-code-correctness).

**LOW** — `hardenBoardEntry` in `internal/loomcli/hardenwiring.go`
- Summary: the production seam is still reached by no test.
- Evidence: the new `TestReadBoardEntry` drives `readBoardEntry` through a fake `boardTaskReader`. `hardenBoardEntry` (which passes `boardengine.OpenHub` and `seedSlug`) and `hardenCampaign`'s `Entry: hardenBoardEntry(location)` are exercised by nothing. `TestHardenPreflight` passes a fake `entry`.
- Impact: dropping the `Entry` wiring in `hardenCampaign` leaves every test green. The review's F2 asked for a test that reaches the seam through `hardenCampaign`.
- Why it is not higher: the refusal text and the present, absent and failure paths of the entry read are covered, so the correctness risk is small.

Walked through with no defect found:
- **`hardenPreflight`**
  - Absent `gate.yaml` returns the `lym fabric reconcile` error, and the tier-0 gate over `loomcli` and `loomrecipe` stayed green.
  - With no packages and a failing `entry`, it returns the entry's error before `CheckCampaign`.
  - With packages named, `entry` is never called.
  - `campaign.Entry` is always set when `wireHarden` builds it.
- **`hardenBoardEntry`**: `return boardengine.OpenHub(...)` into `(boardTaskReader, error)` yields a non-nil interface wrapping a nil `*Board` on error, but `readBoardEntry` checks `err` first, so it never dereferences it.
- **`CheckCampaign`**
  - Two-tier `harden-live` is refused only when copies run. With LLM driving or concurrency 1 it passes, and the new rows pin both.
  - `gatecli` accepts `--gate harden-live --copies N` with targets for a one-tier key below tier3 (`internal/gatecli/run.go:398-401`, `:450-453`). The fold is sound, and a hub `gate.yaml` that still holds `harden-live-copies` loads, since unknown gate keys load.
- **`NewLiveGate` and `TestLiveGate`**
  - The copies call is `Run(LiveGateKey, concurrency, packages)`.
  - The fake treats `copies > 0` as the copies call. If the merge-bar call ever carried copies, the recorded keys would stop matching, so the test still fails with that defect in place.
- **`RoundFacts`**
  - The `Charter` and `Operator's ask` lines name `round-N-focus.md` and `DecisionRecordPath`.
  - `TestCampaignRoundFactsNameTheCharterAndTheOperatorsAsk` fails without them, for rounds 1 and 2.
- **`workWithInstructions`**
  - Blank instructions leave `work` unchanged.
  - No other recipe's `BurlerFix` row carries `target.instructions`: the `instructions` keys in the motif, sampler, tapestry, arras and darn recipes sit under review rows. Other fixers' prompts are unchanged.
  - `TestComposeFixPrompt_TargetInstructions` fails if the instructions are dropped or come before the work statement.
- **F7 test**
  - Dropping `seed_stencil` makes the seed read the missing `bouncer-template-seed`, so no spec is recorded.
  - Dropping `SeedBrief` leaves the `seed_brief` marker unfilled, so the fill fails and no spec is recorded.
- **F11 assertion**: it passed at tier 1.

Calls used:
1. `sed` of `internal/gatecli/run.go:392-408` and `:440-460`, plus a grep of the recipes for `BurlerFix`, `target:` and `instructions:`. My context lacked `gatecli`'s `--copies` refusal rules and which recipes' fix rows set target instructions.
