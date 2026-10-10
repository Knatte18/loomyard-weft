---
verdict: CIRCLING
rationale: "F2 (BLOCKING design, D8 'only copy' premise false) maps to d8-proc-probes-duplicated and F3 (MEDIUM design, commondir target not checked) maps to d12-walkup-diverges-from-show-toplevel; both keys are open from round 2 per the facts and still open in round 3, so gating findings sit on earlier-open keys. F1 (BLOCKING design, --git-path hooks) is a new key."
---

Round 3 carries three gating-class findings.

- F1 is BLOCKING and new: D12 would drop the `rev-parse --git-path hooks` spawn that honours `core.hooksPath`.
  It is mapped to the new key `d12-hooks-path-spawn-removed`.
  It blocks on its own, but it is not circling.
- F2 is BLOCKING and maps to `d8-proc-probes-duplicated`, open since round 2 (filed there as `scope`, now `design`).
  Round 2's fix left the `loomcli` and `orchcli` copies of the watchdog recognizer and `/proc` probes in place, so the key is unchanged.
- F3 is MEDIUM and maps to `d12-walkup-diverges-from-show-toplevel`, open since round 2.
  Round 2's fix pinned two git cases but not a gitfile whose `commondir` target is missing.

No finding is relabeled.
The round 2 keys `d12-reader-test-import-cycle`, `d12-cwd-resolution-constraint-understated`, `census-drop-column-unnamed` and `test-budget-list-incomplete` were each cleared by the round 3 focus verdicts, so they are resolved.
