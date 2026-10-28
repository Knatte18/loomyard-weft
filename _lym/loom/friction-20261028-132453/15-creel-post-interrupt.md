# Batch 15: a creelengine daemon-lifecycle test failed once with no cause in the batch

The card 17 gate failed once on `TestDaemon_Lifecycle/HookWaitReconnectsAcrossARestartAndAPostToTheNewDaemonRingsIt` with "the daemon ended before it listened: creel: another daemon holds this hub" (`server_test.go:1077`).
Card 17 changed only prose and doc comments, and the same gate passed on the next run, as did card 15's run and the batch gate, so it reads as a race between the first daemon's release of the hub and the second one's start.
Nobody should chase it as a card 17 regression; it may deserve a look as a flaky test.
