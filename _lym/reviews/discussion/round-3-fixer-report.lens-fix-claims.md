```yaml
lens: fix-claims
outcome: reported
```

I checked the fix's added and rewritten factual claims in `_lym/discussion/decision-record.md` (D1, D4, D12, the acceptance criteria and the Notes) against the rollouts and the code at HEAD.

**Claims checked:**
- **D1, the probes:** `awaitLiveJudge` and `awaitLiveSeed` set only `OutputFiles`, `Role`, `Segment` and `Round`, and go to `AttachIfLive` (`bouncer.go:909-916`, `937-944`). Holds.
- **D1, inline prompts:** `runSeedSpawn` and `judgeCall` fill their prompt inline with `stencil.Fill` or `FillOptional` and pass `Prompt: string(prompt)` (`bouncer.go:1722-1729`, `1877-1883`). `runSeedSpawn` passes it to `Attach` and then `Run`. Holds.
- **D1 and D4, a literal that sets no `Prompt` launches nothing:** besides the two probes, the only production literals without `Prompt` I found are the empty `shuttleengine.Spec{}` values returned on errors, for example in `internal/samplerengine/spec.go`, `landingshed/prsession.go`, `mergeresolve/spec.go` and `frictionengine/spec.go`. None of those launches anything. The tackler and arbiter attach probes reuse a launch builder whose literal sets `Prompt`. Holds.
- **D12, `function_call` with a matching `function_call_output`:** the `spawn_agent` at rollout line 50 and the output at line 53 share `call_id` `call_8ae898e2…`. Holds.
- **D12, the Facts:** 21 `exec` items, 2 `spawn_agent` and 2 `send_message` calls in the `collaboration` namespace, and the namespace lists in `additional_tools` all match the rollout. Across the sibling rollouts there are further `spawn_agent` and `send_message` calls. Holds.
- **D12, `CommandExecution` carries `cwd`:** true in both the 11/15 lym rollout and the 11/16 spike rollout, as a `file://` URL. Holds.
- **D12, `cwd` in the payload:** `payloads.jsonl` line 1 has `cwd`. Holds.
- **D12, `parsed_cmd` gives the path as written:** the spike shows `notes.txt`. Holds.
- **D12, no spike has run a `collaboration` call under `PostToolUse`:** the twill spike report never mentions `PostToolUse`, and the 11/16 spike ran only `exec` cells. Holds.
- **D12, the read-again timing:** "the cell's `custom_tool_call` line is not older than the file's last modification" matches `sliceBreach`'s `!info.ModTime().After(messages[i].at)` (`batchingnudge.go:543`). Holds.
- **D12, Claude's matcher leaves an `Agent` call unjudged:** `batchingNudgeMatcher` is `Read|Grep|Glob|Bash|Edit|Write` (`batchingnudge.go:88`). Holds.

**Findings:**
- **LOW.** D12's "Every other model call, such as the `collaboration` namespace's `spawn_agent` and `send_message`, is a `function_call` item…" claims more than the evidence shows. The cited rollouts show `function_call` items only for `spawn_agent` and `send_message`. No call to `wait`, `request_user_input`, `request_user_input_async`, `sleep`, `followup_task`, `interrupt_agent`, `list_agents` or `wait_agent` appears, so their item shape isn't observed. Nothing contradicts the claim, but it is stated as fact. Scoping it to the observed calls would make it accurate, for example "the `collaboration` calls the rollouts show are…".

**Tool calls:**
1. Grepped the 11/15 rollout for `CommandExecution` `cwd` and the twill report for `PostToolUse`: the context only had `cwd` from the 11/16 spike, and twill's `PostToolUse` coverage was unchecked.
2. Read `bouncer.go:1686-1895`: the context held only lines 1-1008, so the inline-prompt claim about `runSeedSpawn` and `judgeCall` was unverified.
3. Ran an awk scan for production `shuttleengine.Spec{` literals without `Prompt`, to check that no launch literal lacks one.
4. Read `samplerengine/spec.go:55-110` and `landingshed/prsession.go:140-165` to confirm that what the scan flagged is the `Spec{}` error returns and not launches.
