```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I traced each change in the diff to the board entry's goal points and checked whether any goal point is left unbuilt.

**Decisions.** The answer key is a decision record alone. It has no `Added after Discussion` entries, so the decisions are the goal points of the board entry under `## Goal` (`_lym/discussion/decision-record.md:3-23`):
- **G1.** Have plan writing or plan validation handle a symbol whose first production caller is in a later card. The entry offers three routes for this:
  - (a) put the declaration and its first caller in one card;
  - (b) list the guard file in the targets of both cards;
  - (c) let the card gate defer the guard until the card that adds the caller.

  "What happened" and "Why it is abnormal" describe the defect G1 has to prevent: a plan that cannot pass its own gates and forces a loosened check. They are not separate asks.

**Per change:**
- **C1:** the new `create-caller-later` check (`internal/planparser/latercaller.go`), wired into `validate` (`internal/planparser/validate.go:354`). Traces to G1 by the plan-validation route; it enforces route (a) by refusing the plan, and its way forward is to move the `Create` or list the caller on the creating card.
- **C2:** `internal/planparser/latercaller_test.go`. Forced by C1 as its test.
- **C3:** the `create-caller-later` entry in `contracts/specs/loom-plan-checks-spec.md:177-188`. Forced by C1, since the spec lists every plan check id.
- **C4:** `contracts/specs/loom-plan-spec.md:363-364`, which adds `(create-caller-later)` to the card-granularity rule. Forced by C1: the rule document now names the check that enforces it.
- **C5:** the redundant-package fixture in `internal/planglyph/glyphchain_integration_test.go:244-245` changes from `func Fresh() int` to `type Fresh struct{}`. Forced by C1. Card 1 creates only the handle, and card 2 names `plan:shapes#Fresh` beside the self glyph `shapes#`, which counts as able to hold a caller. Left as a `func`, the fixture would now raise `create-caller-later` on card 1.

**Per decision:**
- **G1:** built by C1, with C2–C5 forced by it. Routes (b) and (c) are not built. The entry offers them as alternatives ("or"), so leaving them out is not a gap; the summary's design decisions give the reasons (`_lym/landing/summary.md`, "A plan check, not a deferred guard").

**Findings:**
- **LOW, class `scope`.** Location: `internal/planparser/latercaller.go:43`, the `cardNamesRef` gate. G1 is built only for a later caller that names the new function's handle among its targets or `Uses`. A plan whose later card writes the first call without naming the handle, for example by editing `internal/dobby/tick.go` alone, still passes both plan gates and fails the creating card's gate, which is the exact abnormality the entry reports. The summary admits this under `## Limits` ("A function a later card calls without naming it in its targets or `Uses` raises nothing"), so it is a disclosed partial build of G1 rather than a hidden one. I could not confirm whether the reported plan's later card named the handle: the provenance note was not reachable. Evidence: `internal/planparser/latercaller.go:43`, `internal/planparser/latercaller.go:165-168`, `_lym/landing/summary.md` `## Limits`, `_lym/discussion/decision-record.md:6-11`.

No change in the diff is unrequested scope, and no commit carries a `Pattern-Departure:` trailer (`git log 4aa85ab3a..HEAD`).

Tool calls:
1. Searched `/home/knatte/Code/loomyard-LYMHUB` for the provenance note `dobby-run-watch-build/01-run-watch-setting.md`, to see whether the reported plan's later card named the handle. My context lacked it, and the search did not find it.
