# drift: Drift-Recheck halts on a capture's provenance file even when the compare command passed

`drift-capture-provenance-halt` · Note · bug, drift, config

Imported from [issue #71](https://github.com/Knatte18/loomyard/issues/71).

Seen: task run cuttings-fitted-rheology (hub Models, diff: lines) blocked at Drift-Recheck after compare_passed: true, with 'the code changed what the expectation does not cover: changed in case "_source.txt" at key "commit" ... at key "captured_at"'. The hub's baseline harness writes _source.txt (commit SHA and capture time) beside the captured cases; it differs in every capture of a new commit, so every task run with an empty expectation halts here.

Workaround: the parent ran 'lym drift amend <slug> --entries' with keys commit and captured_at of case _source.txt, then 'lym loom resume'. Every drift run in the hub needs the same amend.

Expected: a way to declare volatile capture files or keys hub-wide in drift.yaml (an ignore list), or the line diff skips them, so provenance never reads as drift.

## From issue #79

Imported from [issue #79](https://github.com/Knatte18/loomyard/issues/79).

**What happened**
Two Drift-Recheck halts at the start of the run:
1. 'drift: the drift current capture command holds {skip_file}, which drift no longer renders', with the way forward to remove it via `lym config drift --set` from the prime. The hub's drift.yaml still carried a placeholder that a newer lym no longer renders.
2. 'the code changed what the expectation does not cover': the only differences were the `commit` and `captured_at` keys in the case `_source.txt`, which record where and when the capture ran.

**Why it is abnormal**
(1) A config key that lym dropped support for was left in an existing hub config, so a run blocks on stale config with no migration. (2) `_source.txt` is lym's own capture provenance and changes on every capture, so any recheck after a new commit reports an unexpected change; `lym drift amend` cannot sensibly expect a value that changes each run.

**What to do**
(1) Migrate or warn about unrenderable placeholders at config load, before a run starts. (2) Exclude the `_source.txt` provenance keys (`commit`, `captured_at`) from the drift comparison.

Provenance: task cuttings-fitted-rheology; notes loom-halt.md, loom-halt-2.md