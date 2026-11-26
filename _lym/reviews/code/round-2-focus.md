---
round: 2
focus:
  - "internal/websterengine/recoverbatch.go (recoverSpawn, awaitTerminal, RecoverSpawnOrAttach), the `lym webster recover-batch` verb in internal/webstercli and contracts/stencils/webster/webster-template-master.md: when a recovery waits on its parent, trace the question and the parent's answer end to end. Does the resumed strand's prompt receive both, and what bounds repeated waits of one batch?"
  - "internal/websterengine/recoverbatch.go awaitTerminal: which turn ends without a report become a waiting digest and which stay dead/asking under the commit-and-count retry rule? Compare each case, a closing summary included, with what the board entry asks, and check the tests pin it."
  - "internal/websterengine/recoverbatch.go awaitTerminal: on every return path, error paths included, is the recovery strand stopped and the in-flight record left as the function's doc comment states?"
  - "The reset and status verbs in internal/webstercli and their internal/websterengine code: how do they treat a failed, dead or stuck batch record that holds an unrendered AmendedCards entry, given that internal/websterengine/rebaseline.go records amendments on such terminal batches?"
---

The last entry covers readers of the batch record that have not been examined yet.
