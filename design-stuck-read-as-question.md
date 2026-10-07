# A Webster stuck outcome is reported as the master asking a question

`stuck-read-as-question` · Note · bug, loom, shuttle

Seen 2026-10-07 in card-batching: the batten notice and loom status said `webster master is asking a question`, but the master had written `_lyx/webster/outcome.yaml` with `outcome: stuck` plus `summary.md` and ended its turn. The stop must be classified from the outcome file, not as a question; see `session-state` for the wider model.