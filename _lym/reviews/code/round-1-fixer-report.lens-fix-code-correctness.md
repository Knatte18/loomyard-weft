```yaml
lens: fix-code-correctness
outcome: reported
```

Task: checked the correctness of the code the fix changed (`fix-code-correctness` lens), by reading it and tracing inputs; I ran no tests.

**1. BLOCKING: a re-entered run cannot resume a waiting recovery batch**
- **Changed text:** the new refusal in `RecoverSpawnOrAttach` (`internal/websterengine/recoverbatch.go:539`) and `resumeStep` (`internal/websterengine/waiting.go`), which now renders `recover-batch N --answer "<the parent's answer>"`.
- **Defect:** `contracts/stencils/webster/webster-template-master.md`, the `waiting` bullet of the resume-trail list, was left unchanged. It still says "The run was re-entered, so treat the step as taken: resume the batch with the verb named after `resume:`".
- **Trace:** a fresh Master holds no parent answer, and this bullet tells it not to ask for one.
  - Running the verb bare is now refused, and no ladder rung covers that refusal. Before the fix, the bare call resumed a NEEDS_PARENT recovery.
  - Running the `resume:` text as printed passes the non-blank check. The strand's prompt then holds `answer: <the parent's answer>` as if the parent had said it.
  - Otherwise Master has to invent an answer.

**2. MEDIUM: the new recovery prefix forbids re-reporting a need that is still unmet**
- **Changed text:** `contracts/stencils/webster/webster-prefix-recovery.md`, "Act on the answer, and do not report the same need again."
- **Contradiction:** the Master template's `waiting` resume bullet says "A strand that still finds the step untaken reports it again, and the waiting rung of the failure ladder below handles that."
- **Trace:** the parent answers "done" but the `gate.yaml` setting is still missing, or the reply only partly answers. The strand is told not to report that need, so it can only report FAILED or end its turn, which makes the batch `dead`/`asking`.

**3. LOW: the failed-batch refusal now ends with the message after its way forward**
- **Changed text:** `internal/webstercli/recoverbatch.go`, `msg := err.Error() + lastMessageClause(result.Digest.LastMessage)`.
- **Defect:** `err.Error()` for a `BatchFailedError` ends in its way-forward steps, so `(its last message: …)` trails the last step and reads as part of that instruction. The strand's free text can itself contain commands or "way forward:".
- **Test gap:** no test the fix added reaches this branch, so deleting the append would pass the suite.

**4. LOW: transient way forwards omit the now-required `--answer`**
- **Changed behaviour:** a waiting recovery's resume needs `--answer`.
- **Defect:** `recoverSpawn`'s HEAD-read, card-hash, uncommitted-paths and strand-start errors still say ``re-run `lym webster recover-batch N` ``. `recover-batch`'s reed-boot refusal says the same.
- **Trace:** a resume whose `Starter.Start` fails saves no state, so the record stays waiting. Following the named step then hits the blank-answer refusal. That refusal names `--answer`, so it recovers after one extra round.

The rest held up in the trace:
- the `awaitTerminal` message read and its warning path;
- the rebaseline `continue`, which restamps before skipping and matches the pre-branch behaviour for failed and stuck batches;
- the `ParentAnswers` carry across recovery spawns;
- the `RenderRecoveryPrompt` callers;
- the new test rows, each of which fails with its defect in place.

Tool calls (1 of 6):
- `sed` of `webster-template-master.md` lines 62-98 plus a `grep` for the waiting and resume sites: my context held the template only from line 100, and not every consumer of `resumeStep` and `waitsOnParent`.
