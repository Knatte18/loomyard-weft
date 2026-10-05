---
round: 1
focus:
  - "decision-record.md D1 Rationale and Auto-mode assumptions: does the claimed `CLAUDE_ENV_FILE` behavior (read once per session, content pushed into the prelude before the eval, same shell) hold as a premise the design depends on, and is each load-bearing claim stated once with its failure mode in Open risks?"
  - "decision-record.md D1 and D2 Bounds: does each stated bound match what the Decision actually introduces (the env file's content, the `--full` flag, the record-first write with stdout fallback), including the case where the operator's own `CLAUDE_ENV_FILE` is overridden?"
  - "decision-record.md D2 against Acceptance criteria: do the success and error key lists in D2 and the acceptance bullets for them agree, and is every key the driver stencil branches on per step present in the short envelope (D3 names what it reads from `envelope_path` instead)?"
  - "decision-record.md D3 `## Drive directory` change: the `(cd <drive-dir> && lyx …)` rule becomes a no-`cd` rule, so what does the Decision say bounds a session whose cwd is not the drive directory, and does Open risks cover it consistently?"
  - "decision-record.md Auto-mode assumption that no Go code parses `lyx shed step` stdout for the removed keys: is it a premise the design rests on, and does an acceptance criterion or note keep it from silently failing?"
  - "Relocation check, decision-record.md to support-log.md: do the D1 Rationale internals-of-Claude-Code detail and the last `Notes for the plan writer` bullet repeat each other, and is any narrative there that the plan writer does not need? Extract any requirement into Decisions or Constraints before proposing a move."
---

Round 1 seed.
The entries name where to look and which question to settle; none pre-states a verdict.
None targets a missing "Notes for the plan writer", missing rejected alternatives in the decision record, or call-site enumeration, per the rubric's `Do not flag` list.
