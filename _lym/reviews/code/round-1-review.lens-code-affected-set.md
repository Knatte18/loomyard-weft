```yaml
lens: code-affected-set
outcome: reported
```

I reviewed what the diff's changes force outside the lines they touch: callers, tests, docs, stencils, rule documents and commit trailers.

**Per change, the set found, covered or not:**

1. **`template.yaml` defaults for `reviewer` and `fixer` change to `["sonnet[high]"]`.**
   - `TestLoadConfig` defaults (`config_test.go:53`): covered by the diff.
   - `doc.go` Config section (`:13-14`) and Method section (`:96-97`): covered by the diff.
   - The judge's convergence rule in `contracts/stencils/harden/harden-rubric-review.md:98`, which needs the reviewer model to differ from the previous round's: **not covered** (F1).
   - The round facts' `Models:` line (`campaign_rows.go:186`) carries the reviewer model the judge compares; that code is unaffected.
   - Hub file `_bolt/_lym/config/hardener.yaml` sets neither key, so the new default reaches it. This matches the summary's claim.
   - No other doc, stencil, PATTERN entry or `pattern/` file repeats `opus, fable, sonnet` or calls rotation the default. A grep of `internal`, `contracts`, `docs` (outside `docs/research`), `PATTERN.md` and `pattern` found none.
   - The `TestCheckCampaign` fixture (`campaign_test.go:30`) still uses `opus, fable, sonnet`. That is a test input, not a default, so it is not wrong.
2. **`reviewerRepeat` now returns `""` for lists of length 0 or 1.**
   - The one caller, `CheckCampaign` (`campaign.go:85`), is covered.
   - The `CheckCampaign` doc comment (`:48`) and the `reviewerRepeat` doc (`:121-122`) are covered.
   - Test row `ReviewerListOfOneEntryPasses` is covered.
   - The old message "holds one entry" survives in no doc or stencil (grep found none). Covered.
   - At runtime, `Rotate` and `cycled` take a one-entry list and repeat it every round (`rotate.go:7-16`, `campaign_rows.go:115-120`). Covered, and `TestRotate`'s `OneEntryRepeats` row pins it.
3. **New `TestPrepareRunSettings` row (lists under `hardener`).** It pins existing behaviour of `PrepareRunSettings` (`runsettings.go:43-79`). Nothing else is affected.
4. **Commit trailers.** `0196d0f69` carries `Pattern-Departure: none` (F2). No `Loosened-Check:` or `Root-Cause:` trailer is present.
5. **Declared PATTERN changes.** The directive declares none, so there is nothing to land.

**Findings:**

- **F1 — BLOCKING, design.**
  - Location: `contracts/stencils/harden/harden-rubric-review.md:98`.
  - Summary: the judge rules CONVERGED only on a round "whose reviewer model differs from the previous round's". With the new default of `reviewer: ["sonnet[high]"]`, which preflight now accepts, every round reviews with the same model. A default campaign can therefore never converge and runs until its round cap.
  - The diff leaves this stencil rule tied to rotation, which contradicts the operator's decision that one model with no rotation is a working default. The decision also asks that "any doc or pattern text" premised on rotation follow it.
  - Evidence:
    - `harden-rubric-review.md:97-99`: "It reads all three from the round's facts."
    - `campaign_rows.go:186` writes the facts line "Models: reviewer `%s`".
    - `template.yaml:5`.
    - `campaign.go:124-126`.

- **F2 — NIT, consistency.**
  - Location: commit `0196d0f69` message.
  - Summary: the trailer `Pattern-Departure: none` names no PATTERN entry, and the commit makes no departure. Under the rubric's "Commit messages and departures", a trailer naming an unknown entry is a finding. The trailer should be dropped.
  - Evidence: `git log` output over `30ed1ee58..HEAD`.

**Tool calls:**

1. Grepped `internal`, `contracts`, `docs`, `PATTERN.md` and `pattern` for literal old defaults or list-valued `reviewer:`/`fixer:` lines, and `docs/*.md` for hardener model text. The manifest's grep covered only "rotation" wording, not other readers of the default values.
2. Grepped the loomcli and shedrecipe harden tests for `fable`, read the hub's `hardener.yaml` for `reviewer:`/`fixer:` keys, and grepped `PATTERN.md` for hardener entries. The manifest held none of these.
