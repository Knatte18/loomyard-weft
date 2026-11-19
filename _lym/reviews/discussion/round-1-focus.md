---
round: 1
focus:
  - "D9, D10, D11, D12 (new server, token, launcher, write routes): for each verb, route, escape hatch or weakened guard, check the Decision states what it lets through and what bounds it. Settle whether the file route `GET /file/<name>`, the unauthenticated `GET /`, the launcher file and the idle and exit paths each carry a stated bound."
  - "D4 link sanitising, D1 JSON escaping, D9 CSP by hashes, D5 `xdg-open` foreground behaviour, D6 VS Code task and workspace `tasks` key behaviour, D2 `fetch` blocked at `file://`: each rests on how a browser, Go or VS Code behaves. Settle whether each states the observed result and where it is pinned (test, recorded command and output, or documentation), or only asserts it."
  - "D5, D9, D10, D11: look for facts left for Plan or an implementer to find out, such as an unnamed bound, 'a fixed byte cap', a wait 'bounded' with no stated value, or behaviour of `runproc`, `fswatch`, `selfexec` or `lymcwd.ResolveWithin` the design relies on with no evidence recorded. Settle whether each is a fact the design rests on or a plan-level value."
  - "D8 and D10 (revision hash, check inside the write section, side-effect writes to siblings): check the bound is complete and consistent with D3's note that unshown fields change `board-data.js`, and that no Decision asks the run to change this task's own board entry."
  - "The three `Added after Discussion` entries that name D7, F7 and F8, against the record's Notes saying D7 is unused and the Open risks section: settle whether any remaining text still depends on a decision or finding that does not exist in this record, and whether the Open risks line is a `decision` for the parent or one an Added entry already answers."
  - "Relocation and leanness: settle whether any Decision rationale, bound or Notes line carries deliberation that belongs in `support-log.md`, applying the completeness-before-leanness test first so no requirement the plan writer needs is lost."
  - "Acceptance criteria against Decisions: settle whether each stated bound and refusal in D3, D5, D6, D9, D10, D11 has a behaviour criterion or a `Review:` marker, and whether the two mechanical test-tier claims in Constraints match the criteria."
---

Seed focus for round 1.
The design is large and new: a loopback server, a token, write routes and a launcher.
The focus points at trust boundaries, unpinned tool facts and the record's stale references.
Nothing here is drawn from the rubric's Do-not-flag list: the record's missing rejected alternatives, absent plan-writer notes and incomplete call-site enumeration are not raised.
