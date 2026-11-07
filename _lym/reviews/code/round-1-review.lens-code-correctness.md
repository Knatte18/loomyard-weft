```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function or branch with a concrete input.

**Per-function list**

- `readLoadTurn`, load message then one `custom_tool_call` and one `custom_tool_call_output`: both recorded, `found` true. Correct.
- `readLoadTurn`, output line before any load message: skipped because `found` is false. Correct.
- `readLoadTurn`, a second load message: `turn` reset to `loadTurn{}`, so calls and outputs share the one boundary. Correct.
- `readLoadTurn`, truncated JSON line: skipped by the unmarshal error. Correct.
- `toolOutputText`, output is a string: returned as is. Correct.
- `toolOutputText`, output is text blocks: joined with no separator. Correct for the real rollout.
  - The real rollout (`rollout-2026-11-06T13-29-27-…jsonl`, line "ordinal":20) has block 1 as `Script completed\nWall time 0.5 seconds\nOutput:\n` and block 2 as the file text.
  - The whole file sits in one block, so the join is harmless.
- `toolOutputText`, output is null, a number or an object: `Unmarshal` into a string fails, `Unmarshal` into `[]textBlock` fails, result "". A null output unmarshals into `[]textBlock` as a nil slice and gives "". Correct, and a "" output never matches non-empty content.
- `toolOutputText`, malformed payload: returns "". Correct.
- `printedFile`, file removed or unreadable: `err != nil`, returns false, so the skill reads missing. Correct.
  - Sequence: `skillPath` found the file, then it vanished before classification.
  - Result: a false "missing" is possible but narrow, and a rare path.
- `printedFile`, empty or whitespace-only file: returns false. Correct. A trailing-newline file is trimmed, so a printed copy without the final newline still matches.
- `printedFile`, whole SKILL.md inside a larger output such as five files in one script output: `Contains` is true for each skill. Correct. This is the real failure case, and the diff settles it.
- `printedFile`, partial output: a string output of `# testing\n` does not contain the full trimmed body. Missing, which is correct.
- `ClassifySkillLoad`, `file == ""`: unknown, and `namesPath` and `printedFile` are not reached. Correct.
- `ClassifySkillLoad`, call names the tail: `namesPath` is true and `printedFile` is short-circuited. Correct.
- `ClassifySkillLoad`, skill dedup: each skill reads its file once per call. Cost is trivial.

**Findings**

1. LOW, design, `internal/shuttleengine/codexengine/skillload.go:259-266` and `:343-351`.
   - The whole-file match assumes the output text carries the file bytes verbatim. It fails for any output that is escaped or wrapped.
   - Cases: a `function_call_output` whose `output` string is a JSON envelope such as `{"output":"…\\n…","metadata":…}`, which older Codex builds wrote; a tool that truncates a long SKILL.md (a "tokens truncated" marker); a CRLF file read back with LF.
   - The skill then still reads missing, which is the old false "not loaded". I did not verify that any current Codex build writes these shapes. The real rollout's `custom_tool_call_output` blocks match verbatim.
   - Evidence: `toolOutputText` unmarshals only a string or text blocks and does no envelope decode. Suggested fix: decode a JSON-envelope string's inner `output` field, or accept it as a known gap and state in the comment that a truncated or escaped output reads missing.

2. LOW, design, `internal/shuttleengine/codexengine/skillload.go:265`.
   - False loaded is possible when a short or generic SKILL.md body is a substring of an unrelated output of the load turn. An example is a `grep` or `ls` result, or the model's own printed text.
   - The old rule needed a call naming the tail. `printedFile` has no tie to a call, only to some output.
   - Likelihood is low because real SKILL.md files hold many lines.

No other defects found in the diff's code paths.

**Tool calls:** none. My context already held the diff, `skillload.go`, the test file and the real rollout excerpt.
