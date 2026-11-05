---
round: 3
focus:
  - "In `_lym/discussion/decision-record.md` D7 and D8, settle whether the baseline capture under `.lym/drift/baseline`, which git does not track, is protected against edits by the writer or judge between the drift gate entry and Drift-Verify, and whether the Decision states what bounds a changed baseline."
  - "In D9, settle whether the judge tree guard covers every place a judge session could change evidence: ignored paths such as the baseline, weft files outside `_lym/drift/`, and a `judgement.yaml` left by an earlier spawn or an interrupted judge; and whether a guard Stuck names the paths and a way forward that survives `lym loom resume`."
  - "In D2, D5 and D11, settle whether `command` mode, which exists for a capture that cannot meet D1's identity contract, still relies on D1 identities for the expectation listing and amendment validation, and whether a resume at an unchanged warp head can re-run a judge until it rules `matches`."
  - "In D3 and D15, settle whether `drift.diff` and `drift.key_patterns` are fixed for the run once the scope is recorded, or are re-read from hub config at later rows."
  - "In D7, D9, D10 and D12, settle for every Stuck the record defines (judge `does not match`, a stored ruling reproduced, an exhausted `judgement` entry, a failed or timed-out current capture, a run whose capture never counted) whether it names the way forward the parent can take, and whether the capture lock and gate slot have a stated acquisition order."
  - "In D4, D12 and D14, settle what a run in flight when this lands does: a live agent of a removed row, a scope record with `WaivedAt` set, a `partial` scope with its `skip` list, and a hub `gate.yaml` entry for `drift-verify`."
---

Entries name where to look; none caps a severity or states a verdict.
