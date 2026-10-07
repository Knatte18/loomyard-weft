---
round: 2
focus:
  - "Webster call sites: `webstercli` verbs that form batches (`beginbatch.go`, `awaitbatch.go`, `recordbatch.go`, `recoverbatch.go`, `validate.go`) and `Run` in `runlevel.go` each hold one batch-forming line. Settle whether any file or call site is edited by more than one card in a way a later card overwrites, so an earlier card has no surviving reviewable content."
  - "Config open maps: `batcher.yaml` has a `profiles:` map keyed by operator-chosen names. Settle whether the plan declares it an open map in `internal/configreg` and passes that to the load, so reconcile does not prune operator profiles and the load does not fill in template profiles the operator removed."
  - "Fork attribution in `tools/tokencount`: real fork transcripts carry the prompt file as a `Read` tool result with a `<number><TAB>` line prefix, and the prompt also holds card gate lines that share the leading shape of pointer lines. Settle whether the attribution card's rule and fixture work against that transcript form."
  - "Test economy across the batcher cards: settle whether any planned `TestActive` or config test row exercises a parse error that another card's test already covers through `ProfileWeights`."
---

Entries stay within the rubric's `Do not flag` list and the writer symmetry rule.
