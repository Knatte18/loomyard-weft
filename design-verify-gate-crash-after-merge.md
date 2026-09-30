Found reviewing PR #308 (`publish-verify-after-merge`).
Fabric reports a finished-but-interrupted merge as already up to date, so the gate cannot tell a verified tree from an unverified one.
Robust fix: record the task HEAD that last passed verify, and verify whenever HEAD differs from it.