```yaml
lens: fix-code-correctness
outcome: reported
```

Lens: fix-code-correctness, over the five fix commits.

**Findings**

- **LOW** — `ErrorEnvelopeLine` in `internal/output/output.go`, fence toggle (`strings.HasPrefix(trimmed, "```") || ... "~~~"`).
  - Summary: the toggle ignores fence length and fence character, so a nested or mixed fence flips the state wrongly.
  - Evidence: the input "````md\n```\n{\"ok\":false,\"error\":\"x\"}\n```\n````\n" runs as follows.
    - Line 1 opens the fence.
    - Line 2 (the inner ```` ``` ````) closes it.
    - The envelope line is then outside a fence and is matched, which is a false positive.
  - The same happens for a "~~~" line inside a "```" block.
  - Related edge: an unclosed fence hides every later line, so a bare envelope after it is missed. `UnclosedInnerFence` is an existing body shape in the `recordwrite` tests.
  - The reported case, an unfenced envelope under a heading, is still caught.
  - The reviewer's proposed toggle has the same limit, so this is minor.

**Traced with no defect**

- **F2 tests.**
  - `EnvelopeInsideAFencedBlock` fails without fence tracking.
  - `BareEnvelopeAfterAFencedBlock` fails without it too: the defect returns the quoted `"quoted"` line, not `"boom"`.
  - The existing `EnvelopeUnderAHeading` row has no fence and is unchanged.
- **F3 rows.**
  - `isBareSlug("1abc")` returns false at index 0 (a digit, not a letter). `isBareSlug("my task")` returns false at the space. Both then reach the JSON decode.
  - Neither is a handle: the handle path needs a letter followed by digits only, and `1abc` starts with a digit.
  - I did not confirm that the decode error text starts with `invalid json:`. `wantError` is a substring match (`cli_test.go:448`), and the review states that is the message.
  - The rows would fail if `isBareSlug` accepted any rune, since the lookup would then return "not found".
- **F1.** The reason string's `%s` count is still one, so `fmt.Sprintf(errorEnvelopeBodyReason, envelope)` is intact.
  - Every literal in the updated `want` list appears in the new constant: `lym batten cancel <slug> --remove`, `lym board upsert --body-file <path>`, `lym loom resume`, `lym batten run <slug>`.
  - The test would fail on the old text, which lacked `--remove`, `lym loom resume` and the start verb.
  - The `<slug>` text in the reason contains no `%`, so it is not misread as a format verb.
- **F4, F5.** Doc and help text only. No code path or caller is affected.

**Further tool calls**
1. A `grep` for `isBareSlug` and the `wantError` comparison in `internal/boardcli`: the bodies of `isBareSlug` and the substring check were not in my context.
