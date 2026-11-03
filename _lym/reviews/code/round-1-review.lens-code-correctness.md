```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran one concrete input through each changed function or branch.

**Per function or branch (case run, verdict)**

- `boardengine.RemoveCommand` (`board.go:225-230`):
  - Slug `fix` gives `lym board remove '{"slug":"fix"}'`. Correct, and it matches the form in `cli.go:689`.
  - Empty slug gives `'{"slug":""}'`. It renders, and the verb would refuse an empty slug. Not reachable from a way forward, so not a defect.
  - Slug `a'b` would break out of the single quotes. `json.Marshal` leaves `'` unescaped, so the output is `'{"slug":"a'b"}'`. I read the pair slug rule only at the `ValidateSlug` seam (`config.go:54`). I did not read the rule itself, so I can't say whether a slug can hold `'`. For ordinary slugs the output is correct.
  - `json.Marshal` of a `map[string]string` can't fail, so the discarded error is correct.
  - `json.Marshal` escapes `<`, `>` and `&` as `<`, `>` and `&`. The remove verb's JSON parser decodes these back, so the form is correct.
- `frontFixedRefusal` (`front.go:16`): slug `fix` gives "...remove it with `lym board remove '{"slug":"fix"}'`, preceded by `lym batten cancel fix --remove` while a run holds the entry". Correct. The old text used double quotes around the command; the new text uses backticks, which avoids nested quote clashes with the JSON.
- `RemoveError.HandRemoval` (`fronttask.go:127-133`):
  - Step `cancel` gives "run `lym batten cancel fix --remove`, then `lym board remove '{"slug":"fix"}'`". Correct.
  - Any other step gives "run `lym board remove '{"slug":"fix"}'`". Correct.
  - The import `boardengine` was already present at `fronttask.go:15`, so there is no new dependency or cycle.
- `fronttask.File` open-front refusal (`fronttask.go:180`): open slug `older` renders both commands. The five-argument format matches its five verbs (`req.Slug`, `open.Slug` ×3, `RemoveCommand`). Correct.
- `judgeOwnFix` drop line (`fixloop.go:268`): the two arguments feed `%s` for the cancel command and `%s` for `RemoveCommand`. Correct. The backticks around the command are balanced.
- `removeFixTask` fallback (`green.go:203`): the format has `%s` for slug and subject, `%v` for err, `%s` for slug, then `%s` for `RemoveCommand`, so there are five verbs for the five arguments listed. Correct. The `errors.As(err, &removal)` branch still goes through `HandRemoval`, which is correct.
- `adoptcli/doc.go:178`: the doc text now names the JSON form. Correct.
- `stepFrontTask` (`boardcli/verbs_test.go:427-434`):
  - `fixed` is built as `lym board remove '{"slug":"fix"}'`, and `refuse` presumably checks the error contains it. That is a real pin on the `frontFixedRefusal` text.
  - `runJSON(..., "remove", removePayload)` then proves the same payload parses and removes the entry.
  - Gap (see F1): the test never feeds the `fronttask` or `adoptcli` rendered commands to the verb. Those packages only compare strings. That is the code-tests lens's domain, so I only note it.

**Findings**

- None at BLOCKING or MEDIUM from the code I ran.
- LOW, `design`, `internal/boardengine/board.go:228-229`: `RemoveCommand` single-quotes the payload without escaping a `'` inside it. If a slug can ever contain `'` (I did not read the slug rule), the rendered command is a broken shell line. Evidence is `board.go:228-229` plus `config.go:54-56`, where `ValidateSlug` is only a seam, so I could not confirm the rule. Fix: confirm the slug rule's character set. If it excludes `'`, add nothing. Otherwise escape `'` as `'\''` in the payload.

Tool calls (1):
1. `grep` for `ValidateSlug` and slug regexes in `boardengine`, to see whether a slug can contain `'`. The manifest lacked the slug rule. The result showed only the seam, not the rule.
