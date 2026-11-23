```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on this diff. I traced each changed function and branch with concrete inputs.

**What I ran**

- **`reviewerRepeat`, empty list (`nil` or `[]`)**: it returns `""`. This case can't reach it from `CheckCampaign`, which refuses an empty reviewer list first (`internal/hardenshed/campaign.go:81-83`). Correct.
- **`reviewerRepeat`, one entry (`["sonnet[high]"]`, the template default)**: `len <= 1` returns `""`, so `CheckCampaign` goes on to the `max_bounces` check and returns nil. Correct, and it matches the added entry.
- **`reviewerRepeat`, two entries with one model (`["opus", "opus[high]"]`)**:
  - At i=0, `next` is `"opus[high]"` and both `modelOf` calls give `"opus"`.
  - It refuses, naming `"opus"`, `"opus"` then `"opus[high]"`.
  - `CheckCampaign` passes that through `refuseKey("reviewer")`.
  - Correct: the refusal for lists of two or more stays.
- **`reviewerRepeat`, two models (`["opus", "sonnet"]`)**: at i=0 opus differs from sonnet, and at i=1 sonnet differs from opus, so it returns `""`. Correct.
- **`reviewerRepeat`, three entries whose last and first match (`["opus", "sonnet", "opus"]`)**: at i=2, `next` is `reviewer[0]`, both are `"opus"`, and it refuses. Correct.
- **Refusal message at every length:**
  - The message text (`campaign.go:130`) is unchanged by the diff.
  - Only lengths of two or more reach it now.
  - For a pair that doesn't involve the wraparound, it still says "its last and first entries included". That reads as a description of the check rather than a claim about this pair, and it predates the diff.
  - Not a defect of the diff.
- **`CheckCampaign`, one-entry fixer list (`["sonnet[high]"]`)**: the fixer gets only the emptiness check, and `reviewerRepeat` looks at the reviewer list alone. It passes. Correct, as before.
- **`template.yaml` `reviewer: ["sonnet[high]"]` and `fixer: ["sonnet[high]"]`**:
  - The quoting keeps `[high]` from being parsed as a nested flow sequence, so each loads as the one-element list `[]string{"sonnet[high]"}`.
  - `TestLoadConfig`'s defaults expect exactly that.
  - `checkValues` parses each entry as a model-spec, and `sonnet[high]` is a valid alias with a parameter.
  - At runtime, `Rotate(["sonnet[high]"], HardMaxBounces)` repeats it every round, and `cycled(list, round)` returns `list[0]` for each round of 1 or more.
  - Correct.
- **`PrepareRunSettings` with `hardener.reviewer` and `hardener.fixer` lists (the new test row)**:
  - `yaml.Marshal` of the map sorts the keys, so `fixer` comes before `reviewer`, and each list keeps its order.
  - `sonnet[high]` is emitted unquoted as a block-sequence item, which is valid because `[` isn't the first character.
  - `checkLocalDocument` accepts both keys, since neither is a guard key or a retired key.
  - The document the row expects is what this code path produces. Correct.
- **`doc.go` comment edits**: no code path. Not in this lens.

**Findings:** none. No wrong result, swallowed or misreported error, or inconsistent failure state in the changed code.

Tool calls: none. The inherited context already held every definition I traced.
