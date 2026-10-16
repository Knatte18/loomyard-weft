---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/seatengine/engine_test.go:181
    summary: tests still use the retired fan_review key as the fanned reviewer's model key
    fix: In internal/seatengine/engine_test.go line 181 (the cluster-fan reviewer row) use a live reviewer key such as discussion_review in both the ModelKey and the wantText, and do the same for the ModelKey literals naming fan_review at internal/shedrecipe/entries_burlerhalves_test.go line 431 and internal/burlerengine/review_test.go line 49.
---

### [NIT] Tests still name the retired `fan_review` key

**Location:** `internal/seatengine/engine_test.go:181`, `internal/shedrecipe/entries_burlerhalves_test.go:431`, `internal/burlerengine/review_test.go:49`

**Issue:** `fan_review` is retired in `loom.yaml` (`ConfigSpec.RetiredKeys`, `retireFanReview`, the template and `refusal-spec.md` no longer carry it).
Three tests still use it as the example model key of a fanned reviewer, and the seatengine row asserts the refusal text `set fan_review in loom.yaml to an alias whose engine meets them`, which no loom hub can follow any more.
The values are inputs, so nothing fails; they only keep a retired key in the test vocabulary, against the focus question on leftover references.

## Review notes

The review range is `git merge-base main HEAD..HEAD`: `_lyx/shed/fan-prompt-tuning/status.json` gives `product.parent` `main`, and `_lyx/loom/rework/` does not exist.
The unit tier and the integration tier of the plan's verify packages pass, plus `cmd/lyx` (test budget) and `internal/seatengine`.
No commit carries a `Pattern-Departure:` trailer, and the constraints list no declared `PATTERN.md` changes.

Focus questions, each settled:

- **Recorded round fans.** `recordRoundFans` is the only writer of `fan`, `fix_fan` and `fans_recorded` (`internal/shedadapters/roundusage.go`).
  `BurlerReviewProducer.Call` calls it after the probe finds no live reviewer and before the first attempt, and a failed write takes the failure exit with nothing spawned.
  The resumed reviewer probe, the fix half (`readRecordedFans`) and the Bouncer (`excludesLenses`, `lensesRunUnderFan`) gate on `fans_recorded`, never on `fan`.
  `acceptedExit` no longer writes `fan`.
  `computeLensFacts` reads `usage.Fan` only to label the facts section; its exclusion answer goes through `lensesRunUnderFan`, which checks the flag.
- **`fan_review` retirement.** No reader, default, schema entry or template line remains; `RetiredKeys` and `retireFanReview` drop it, and only the three test literals in F1 name it.
  The template defaults are `""` for all four fan keys, so every round runs solo with the single fixer.
- **Byte identity.** The solo explore step is pinned against the explore stencil with `{{.fork_reading}}` removed, and the solo review step against the review stencil filled with today's solo line.
  `{{.tool_use_rules}}{{.fork_reading}}` and `{{.self_check}}## Fixer-report rule` add no byte when empty.
  Fanned rounds keep the lens order through `lensList`.
- **Twill fixer.** `fixTable` sets `ForkSubagents` exactly when `FixFan` is non-empty and keeps the model key zero.
  `forkViolation` keeps `auditClusterRound`'s order and messages, `auditFixRound` makes a wrong count a warning, and `concludeFix` audits before the gate check.
  `twillEngineRefusal` carries exactly one `way forward:` for both key shapes.
  `lyx burler run` reads `fix-fan` from its profile and sets `FixKey` to the default engine and `FixFanKey` to `fix-fan`.
- **Bouncer.** `excludesLenses` is true only when the recorded round fan is non-empty and equals the next round's resolved fan, which settles both the same-fan and the fan-change direction.
  A missing, flagless or undecodable record counts as solo with a note.
- **Per-card checks and test economy.** The `Create`, `Edit` and `Delete` targets of cards 1 to 8 are present (`fannedReview` is gone, every `Create` symbol exists).
  The new tests extend the existing tables and keep to the plan's rule on static prose: they pin the lens order and the presence of rendered parts, not stencil text.
  I found no duplicate coverage.
