---
round: 1
focus:
  - "D3 vs D8, `decision-record.md`: a turn end with every output file present and only unawaited payload-reported shells outstanding finishes Done in the wait loop (D8), while D3 reads any turn end with outstanding background work as `busy`; settle whether D5's disagreement mapping and the acceptance criteria state which wins, or whether this reads as a designed disagreement or a contradiction."
  - "D3, `decision-record.md`: the reducer needs the run mode (autonomous or interactive), the output-file list and an interrupt marker from the transcript, but the decision names 'three facts read at call time'; settle whether each input has a named source and the count and list agree."
  - "Attack surface, D11: the python deny is a regex over the JSON payload with a per-repo off switch; settle whether the Decision states what it lets through (wrappers such as `py`, `uv run`, `bash -c`, `sh -c`, shell-quoted forms) and what bounds that, beyond the Open risks line."
  - "Attack surface, D8: a payload-reported shell now holds a run to its Timeout; settle whether the Bound covers the Master case with `master_timeout_min` and how a hung shell is visible, and whether the turn end that lists it stays free of any parent notice."
  - "Attack surface, D6 and D1: the new `lyx shuttle state` verb and the four new hooks write or read run files; settle whether each Decision states what it can skip or expose and what bounds it, in particular D1's hook-side `lyx_at` being agent-writable input to a reducer that gates no decision."
  - "Relocation: check D9's rationale (commit times, which run predates which) and the Rationale paragraphs that narrate history rather than carry a requirement; per the completeness-before-leanness test, extract any requirement into Decisions or Constraints first, and move only the narrative to `support-log.md`."
  - "Consistency between `decision-record.md` and `support-log.md`: Question ledger Q1-Q15 against Decisions, Auto-mode assumptions and Open risks (Q14 says `StopFailure` stays schema-derived, the llm-tier acceptance criterion lists three payloads); settle whether any ledger answer differs from the Decision it resolved."
  - "Acceptance criteria against Decisions: settle whether every Decision's observable behavior has a criterion, and whether any criterion asserts something no Decision introduces (for example D10's 'byte-identical' claim for existing denies' hooks and notices)."
---

Seed focus for round 1.
Entries were checked against the rubric's `Do not flag` list: none asks for rejected alternatives in `decision-record.md`, a "Notes for the plan writer" subsection, call-site enumeration, or signatures and `file:line` citations that the writer's stencil tells it not to gather.
