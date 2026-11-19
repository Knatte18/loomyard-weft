```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. For each function and branch the diff adds or changes, I ran one concrete input through it.

**Cases run**

- **`haltnote.Render`, empty `RoutedTo`** (`{P, done, T}`): writes `- P / done / T\n`, the same as before the diff. Correct.
- **`haltnote.Render`, `RoutedTo` set** (`{Code-Bouncer, stuck, T, Code-Burler-Review}`): writes `... / T / routed to Code-Burler-Review`. Correct.
- **`haltnote.Render`, empty `At` with `RoutedTo` set:** writes `- P / stuck /  / routed to X`. `parseRow` reads it back as `P`, `stuck`, `""`, `X`. Correct.
- **`haltnote.Render` on a reroute entry** (`Outcome: reroute`, `RoutedTo` set by `internal/shedengine/reroute.go:59`): writes `- P / reroute / T / routed to X`. The suffix is right for this outcome too.
  - Only two kinds of entry carry `RoutedTo`: a Stuck routed through `OnStuck` (`internal/shedengine/run.go:459` and the call 60 lines into the 400–470 span) and a reroute.
  - A Stuck that stops the run (Halt, no `OnStuck`, budget exhausted, reroute cap) passes `""`, so it gets no suffix. The output is correct, but see F2 for the doc comments.
- **`haltnote.parseRow`, old three-field row** (`- A / done / T`): `SplitN(...,4)` gives 3 parts, so `RoutedTo` stays empty. Correct.
- **`haltnote.parseRow`, row with fewer than three fields** (`- A / done`): rejected, as before. Correct.
- **`haltnote.parseRow`, fourth field without the `routed to ` prefix** (`- A / done / T / extra`): `RoutedTo` is set to `extra`. That is tolerant, not a defect.
- **`haltnote.parseRow`, `RoutedTo` containing ` / `:** `SplitN(...,4)` keeps the rest in the fourth part, so it round-trips. Producer names and RFC 3339 `At` values never contain ` / `. Correct.
- **`reportengine` reading `haltnote.Parse`** (`internal/reportengine/stops.go:137`): it reads only `Producer`, `State`, `Reason` and `Anomaly`, so the new field and row shape do not affect it. Correct.
- **`loomcli.writeHaltNote`:** copies `h.RoutedTo`. A history entry from an older status file decodes `RoutedTo` as empty, so its row has no suffix. Correct.
- **`burlerengine.quotingHint`, colon case:**
  - Input: `summary: the refusal prints a creel: warning: line`. yaml.v3 fails with `mapping values are not allowed in this context`, raised only at `scannerc.go:1411` in v3.0.1.
  - The quoted-summary regex does not match, so the colon hint is appended. Correct for that input.
- **`burlerengine.quotingHint`, both a quoted summary and a colon error:** wrong hint. This is F1.
  - Input: `summary: "the gate fails"` (fully quoted, valid) followed by `    fix: print the creel: warning: line`.
  - The `malformedQuotedSummary` regex matches, so the reviewer gets the "quoted fragment followed by trailing prose" hint, and the colon hint never shows.
- **`burlerengine.quotingHint`, type error** (for example a list where a string belongs): the error does not contain the colon text and the regex does not match, so the result is `""`. Correct.
- **`burlerengine.quotingHint`, nil `parseErr`:** it is only called when `err != nil` (`verdict.go:118-119`). No case to run.
- **`claudeengine.isTopBoxRule`, the halted pane's capture** (` ly:...code-burler… ─` above `❯`): now counts as the top rule, so the box reads as empty. Correct.
- **`claudeengine.isTopBoxRule`, cut label with no rule glyph:** still not a top rule. Correct.
- **`claudeengine.isTopBoxRule`, draft line ending `… ─` below the caret:** the bottom rule is found with `isBoxRule`, which does not change, so such a line cannot close the box.
- **`claudeengine.isTopBoxRule`, line ending `… ─` above the caret when a real top rule sits directly above the caret:** the upward walk stops at the real rule first, so it has no effect. Correct.

**Findings**

- **F1. MEDIUM, design.** Location: `internal/burlerengine/verdict.go#quotingHint`.
  - Summary: the quoted-summary branch takes precedence over the new colon hint and matches valid headers, so a colon error is reported with the wrong hint.
  - Cause: `malformedQuotedSummary` (`verdict.go:266`) is `(?m)^\s*summary:\s*"[^"]*"\s*\S`. In RE2, `\s` matches `\n`, so a fully quoted `summary: "x"` matches through the next line's first character.
  - It also matches the stencil's own escaped form, `summary: "\"capital\" is ..."`: the pattern `"[^"]*"` closes at the escaped quote and `\S` matches `c`.
  - Effect: when the summary is quoted, which the new stencil rule now tells reviewers to do, any colon error in `fix:` or another field gets the summary-fragment hint, and the new colon hint is never shown.
  - Evidence: `verdict.go:266`, `verdict.go:275-280`, `contracts/stencils/burler/burler-review-format.md:52`.
  - Fix: keep the match to one line and leave escaped quotes out, e.g. `(?m)^[ \t]*summary:[ \t]*"(?:[^"\\\n]|\\.)*"[ \t]*[^\s#]`. Add a `TestParseReview` row with a quoted summary and a colon in `fix:` that expects the colon hint.
- **F2. LOW, consistency.** Location: `internal/haltnote/haltnote.go:21`, plus the `RoutedTo` field comment at `:50-51`.
  - Summary: the `routedPrefix` comment says the fourth field is the producer "a routine Stuck handed the run on to". The `RoutedTo` comment also casts the field as telling a Stuck hand-off from a stop.
  - In practice, reroute entries (`Outcome: reroute`, `internal/shedengine/reroute.go:59`) carry it too and render with the suffix.
  - Evidence: `internal/shedengine/status.go:106`, `reroute.go:59`, `run.go:459`.
  - Fix: word both comments for any step that routed the run on, a routed Stuck or a reroute.

Tool calls:
1. `grep appendHistory( internal/shedengine/run.go` plus the routing branches in lines 400–470, to confirm which Stuck entries carry `RoutedTo`. My context held only the definition of `appendHistory`, not its call sites.
