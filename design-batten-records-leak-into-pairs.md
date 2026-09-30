Seen 2026-09-30 starting three `lyx batten run` in a row: the 2nd and 3rd blocked at Seed-Child with "task worktree already seeded with a disagreeing seed: run self is already seeded with {Recipe:batten ...}; refusing to overwrite with {Recipe:loom ...}".
Every new pair also carries every other in-flight batten slug under `_lyx/shed/`.
The hub unblocked them by deleting the inherited seed.json/status.json in the task worktree.

Fix direction: give batten its own namespace in the prime (e.g. a batten-owned dir rather than `_lyx/shed/<slug>`), so it can never collide with the child run segment, and make Worktree-Create (or fabric add) start a pair without other runs records.
Also covers `batten-prime-lock-wait` (parallel creates block on the prime lock).