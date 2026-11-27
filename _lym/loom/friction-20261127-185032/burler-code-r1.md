# Fixing the Code-Review round 1 findings of the parallel-tests change

Two `fix` keys could not be applied as written, and each cost a failed gate run.
F5's `SharedHub` switch poisons the shared pair template, because the test it names writes into the hub through a read lock.
F8's `pairHubShape` is declared in `package loomcli`, while the file it names is `package loomcli_test`.
