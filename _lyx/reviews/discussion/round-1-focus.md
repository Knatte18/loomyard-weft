---
round: 1
focus:
  - "Constraints, `PATTERN-mutation-record` bullet, against D4: it says the envelope key set is unchanged apart from `code_push_skipped`, while D4 renames four envelope keys. Settle whether the two statements agree and which one the plan writer would follow."
  - "D2 and D3 attack surface: for the remote deletion in `rollbackAdd` and the prime's records-only push, check that each Bound states what it can still skip or let through, and that the Bound matches the Decision text (adopted vs forked, lease tip, prime-name resolution failure)."
  - "D1 retry classifier: check that the Decision pins what counts as a bare `(failed)` unambiguously enough to separate it from hook-message rejections, and that the Bound covers a push that partially updated the remote ref."
  - "Acceptance criteria against Decisions: check that every behaviour a Decision introduces or changes (D1 through D6, including the D5 `.weft` helper reached from `internal/hubgeom` and the `RefScanner` retired spellings) has a criterion, and that no criterion asserts something no Decision states."
  - "Scope against Decisions and Constraints: check that In/Out list entries do not contradict a Decision (for example the Out bullet on anchored pushes against D3's change to `PushPairAnchored`) and that each Constraint binds a Decision it names."
  - "Relocation pass: for any deliberation narrative in Rationale, Auto-mode assumptions or Open risks that the plan writer does not need, decide whether it belongs in `support-log.md`; apply the completeness-before-leanness test before proposing a move."
  - "Open risks: check that the stranded `side=warp` trace risk and the `batcher-v2` rename conflict carry a stated handling or bound where a Decision relies on them, rather than only a description."
---

Seed focus for round 1.
No entry targets the three `Do not flag` items or enumeration of call sites, signatures or file lines, which the discussion stencil tells the writer not to gather.
