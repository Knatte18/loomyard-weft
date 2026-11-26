```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the fix change set against its siblings and cross-references.

1. **BLOCKING**: the `--answer` requirement versus the Master re-entry rule (`contracts/stencils/webster/webster-template-master.md:80-82`, left unchanged).
   - A resumed Master is told: "The run was re-entered, so treat the step as taken: resume the batch with the verb named after `resume:`, … or a `recover-batch`".
   - For a waiting recovery, `resumeStep` now renders `lym webster recover-batch N --answer "<the parent's answer>"`. A freshly re-entered Master holds no answer.
   - It can only run the placeholder verbatim, which passes the blank check and is rendered into the prompt as the parent's answer, or invent an answer, or drop the flag and be refused. No rung covers that refusal.
   - "A strand that still finds the step untaken reports it again" assumed that resuming without parent input was safe.
   - The same bullet still says only "a step only your parent can take", not a question.

2. **MEDIUM**: transient way-forwards on the resume path still name `recover-batch N` without `--answer`.
   - The sites are in `internal/websterengine/recoverbatch.go`: the HEAD read, card hashes, uncommitted paths, strand start, `renameTool`/pattern-changes and later-delete reads. The reed-boot refusal in `internal/webstercli/recoverbatch.go` does the same (`re-run \`lym webster recover-batch %02d\``).
   - On a waiting recovery, nothing is saved when these fail, so the batch stays waiting. The named re-run is then refused by the new blank-answer check, so the step cannot be carried out as written.

3. **LOW**: sibling docs still define waiting and `Need` as a step only, although a question now takes the same route.
   - `internal/runplan/digest.go`: `DigestStatusWaiting` says "stopped on a step only the run's parent can take", and `Need` says "the step only the parent can take".
   - `internal/runplan/state.go`: the `Status` field doc says "waits on a step only the run's parent can take".
   - `internal/websterengine/waiting.go`: the `waitsOnParent` doc says the same.
   - `webster-body-implementer.md` is composed into the same recovery prompt as the new question section. It still says `need:` is "one line naming the step only your parent can take; NEEDS_PARENT only".

4. **LOW**: the `RecoverDeps.Answer` doc (`recoverbatch.go`, around line 100) misstates the behaviour.
   - It says "every other spawn or attach ignores a blank one", which implies a non-blank answer is used elsewhere or refused.
   - In the code, attach and non-waiting spawns ignore any answer. `recoverSpawn` also records a non-blank answer for a waiting fork batch (`waitsOnParent(prior)` has no `Kind` test).

Tool calls (3 of 6):
- grep for waiting/resume text across contracts and the CLI/engine packages: the context lacked the other places that describe resuming.
- grep for re-arm and `dead_reason` siblings, plus `sed` of master template lines 60-96: the context held only lines 100-189 of that template.
- `sed` of `internal/webstercli/rebaseline.go` help and grep of weaver and `recover-batch` way-forwards: the context had not shown them. No finding came from the rebaseline help or weaver.
