# gitkit and hubforge initialisers write their per-repository git config from Go

`git-config-from-go` · Task · enhancement, fabric, testkit, git · running · Darn-Write

Decision record: [Sketch](decision-git-config-from-go.md)

Filed from the Sketch `fabric-test-speed`.

Takes: gitkit's initRepo and initBareRemote, and hubforge's initScratchRepo and initBareRepo
Delivers: one Go write of today's keys and values into each new repository's config after git init, in place of one git config process per key
Exports: a gitkit helper that writes the keys, which hubforge calls

The part changes these packages:

- internal/gitkit (about 90 changed lines)
- internal/hubforge (about 60 changed lines)
