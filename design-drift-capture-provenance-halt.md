# drift: Drift-Recheck halts on a capture's provenance file even when the compare command passed

`drift-capture-provenance-halt` · Note · bug, drift, config

Imported from [issue #71](https://github.com/Knatte18/loomyard/issues/71).

Seen: task run cuttings-fitted-rheology (hub Models, diff: lines) blocked at Drift-Recheck after compare_passed: true, with 'the code changed what the expectation does not cover: changed in case "_source.txt" at key "commit" ... at key "captured_at"'. The hub's baseline harness writes _source.txt (commit SHA and capture time) beside the captured cases; it differs in every capture of a new commit, so every task run with an empty expectation halts here.

Workaround: the parent ran 'lym drift amend <slug> --entries' with keys commit and captured_at of case _source.txt, then 'lym loom resume'. Every drift run in the hub needs the same amend.

Expected: a way to declare volatile capture files or keys hub-wide in drift.yaml (an ignore list), or the line diff skips them, so provenance never reads as drift.