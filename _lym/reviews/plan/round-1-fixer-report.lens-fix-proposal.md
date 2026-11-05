```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each reviewer proposal the fixer carried out as proposed and build a case where it is wrong or breaks something elsewhere in the artifact.

**LOW — card 13, the `validateLabels` way-forward sentence (F2's fix).** For a discussed note that has lost its record file, the new way forward points to a command that fails at the next step.
- The changed text says: "When `prior` is a discussed note, the write is a Sketch's promotion, and the refusal names `lym board promote <slug> --recipe raddle`". This keys on the `discussed` flag alone, but `checkDiscussedPromotion` keys on "flagged discussed with its record stored".
- Concrete case: a discussed note carries the `sketch` label but its `decision-<slug>.md` file is missing. Card 13's own `TestDiscussedPromotion` already has a row for this state ("a discussed note whose record file is missing (not judged)").
  1. A plain promote is refused by the label rule, which names `--recipe raddle`.
  2. Following that advice produces a raddle task with no stored record.
  3. Card 24's `raddleStartRefusal` then refuses its start ("an entry with no stored record ... run its Discussion first with a motif run").
- Suggested change: name the raddle promote only when `prior` is discussed and its record is stored. Otherwise name the upsert or dropping the label. Alternatively, state that this path is a two-step lead.

Proposals checked with no breakage found:
- **F1:** `boardengine` importing `shedrun` adds no cycle.
- **F4, F5, F6, F7, F9, F10, F11, F13, F14, F15, F16, F17, F19, F20:** no breakage found.
- **F12:** `docs/overview.md` already has a loom config key list (line 637) and a report module line (line 479), so card 27's added step can be carried out.
- **F16 (open question):** a hand-off that finds no orch strand recorded now halts. I could not confirm whether a hub can bring a beam up with no orch. I found no switch for it in `beamup.go`, so I raise no finding.

Tool calls:
- `go list -deps` on `shedrun` and `boardengine`: my context had no import graph to check F1's no-cycle claim. Result: `shedrun` does not import `boardengine`, and `boardengine` already depends on `shedrun`.
- grep of `docs/overview.md`: card 27's added step needs the overview's loom key list and report module line, which my context did not show.
- grep of `internal/pawlengine/beamup.go` for an orch on/off switch: I needed to know whether F16's halt-on-posted-false can fire on every beam. No match.
