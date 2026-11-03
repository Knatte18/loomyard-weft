```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: per behaviour, the test that pins it, or none**

- `boardengine.RemoveCommand` (`board.go:225`) renders `lym board remove '{"slug":"<slug>"}'`.
  - It has no direct test and no table row.
  - Its output is pinned only through the callers below.
  - No test covers a slug containing `'` or a character `json.Marshal` escapes.
- `frontFixedRefusal` (`boardengine/front.go:16`) names the JSON removal form.
  - Pinned by `stepFrontTask` (`boardcli/verbs_test.go:~427`), which runs `refuse(t, fixed, ...)` with `fixed = lym board remove '{"slug":"fix"}'`.
  - The same step then runs the real `remove` verb with that payload and checks the entry is gone, so this is the one place a rendered payload reaches the verb's parser.
  - It runs through the CLI, so the check holds as far as `RemoveCommand` is concerned.
- `RemoveError.HandRemoval` (`fronttask.go:127`), cancel step and entry step.
  - Pinned by the `TestRemove` rows `wantHand` (`fronttask_test.go:276` and `:285`), which assert the JSON string.
  - The rows only compare strings; none feeds the payload to a parser.
- The one-open-front-task refusal in `fronttask.File` (`fronttask.go:180`).
  - Pinned by the `TestFile` row `OpenFrontInProjectRefusesBeforeAnyWrite` (`fronttask_test.go:152`), string only.
- `judgeOwnFix` drop line (`adoptcli/fixloop.go:268`).
  - Pinned by `TestFailedEnd` (`fixloop_test.go:257`) and its `mentions` assertion, string only.
- The `failedEnd` entry-removal-failed line, which uses `HandRemoval`.
  - Pinned by `fixloop_test.go:362`, string only.
- `greenEnd` hand removal after a failed entry step.
  - Pinned by `green_test.go:390`.
  - Before `removeFixTask`'s fallback, the fake's `removeEntryErr` (`green_test.go:134`) surfaces as a `RemoveError` through `fronttask.Remove`, so it takes the `errors.As` branch and `HandRemoval`.
- `green.go:203` is the `removeFixTask` branch for an error that is not a `RemoveError`; it now calls `RemoveCommand`.
  - No test in `green_test.go` sets a non-`RemoveError`, so the changed text has no covering row.
  - A grep for `was not removed` in `green_test.go` found nothing.
  - I did not trace whether `fronttask.Remove` can return a non-`RemoveError` at all.
- `adoptcli/doc.go` names the JSON form. This is static prose, so it is exempt.

**Findings**

- **F-T1: MEDIUM, class `scope`.**
  - Location: `internal/adoptcli/fixloop_test.go:257`, `internal/adoptcli/green_test.go:390`, `internal/fronttask/fronttask_test.go:152`, `:276` and `:285`.
  - Summary: the entry's Expected says a test pins that each way-forward command these packages render parses, but only the boardengine refusal is checked against the verb's parser.
  - The adoptcli and fronttask way-forward commands are asserted as literal strings and never reach the remove verb's parser.
  - Fix direction: since those packages render through `RemoveCommand`, one table-driven test in `boardengine` could feed the extracted payload from `RemoveCommand` to the remove verb's lookup parser.
  - That test would cover every way forward at once, instead of repeating it per package.
- **F-T2: LOW, class `scope`.**
  - Location: `internal/boardengine/board.go:225`, `RemoveCommand`.
  - Summary: no test of the exported helper, and no row for a slug that needs escaping (single quote, `"`, `\`).
  - The discarded `json.Marshal` error and the single-quote shell-quoting are unexercised.
  - This could be one table-driven test on `RemoveCommand`.
- **F-T3: LOW, class `scope`.**
  - Location: `internal/adoptcli/green.go:203`.
  - Summary: the changed fallback branch (a non-`RemoveError` from `removeFixTask`) has no covering row in `green_test.go`.
  - Whether the branch is reachable is unchecked.

**Tool calls**
1. Grepped `green_test.go` and `green.go` for `was not removed`, `removeEntryErr`, `removeErr` and `errors.As`, to check whether the `green.go:203` fallback branch has a test.
