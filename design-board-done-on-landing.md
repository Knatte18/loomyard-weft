Repro: `lyx board upsert` a task, `lyx fabric add <slug>`, `lyx loom start --no-attach` until `state: done` with the change landed on the parent; `lyx board get <slug>` still shows no status (no production code outside `boardcli` sets a board status).
Impact: the board keeps listing landed work as open and claimable, so the operator must `lyx board set-status` by hand after every landing.

Found in crucible-happy-path (round opus-medium-r1).

Source: https://github.com/Knatte18/loomyard/issues/275 (close it in the landing commit).