```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the Goal points of the decision record.

**Decisions** (the record has only the `## Goal` board body and no `Added after Discussion` entries):
- **G1:** a hardener round's agent runs the mutating `lym fabric` verbs in a hub under its own sandbox root, `&lt;worktree&gt;/.lym/harden-sandbox`.
- **G2:** the refusal stays everywhere else.
- **G3:** the guard checks the hub the verb resolves, not only the strand name.

**Changes and what they trace to:**
- **C1** (`internal/fabriccli/roleguard.go`: `sandboxRootRel`, `verbTargetDir`, `inOwnSandboxRoot`, and the new pass in `sandboxedRoleRefusal` at lines 130–132): builds G1 and G3. The pass sits after the reader and merge-task branches, so G2 holds outside the sandbox root.
- **C2** (refusal text at `roleguard.go:133`): forced by C1, because the refusal now has to name where the verb would pass. Serves G2.
- **C3** (`internal/fabriccli/clone.go:182-193`, `cloneDestination` extracted and called at line 222): forced by C1. G3 needs the guard and the clone to resolve the same `--into` destination.
- **C4** (`roleguard.go` file header and `sandboxedRoleRefusal` doc comment): forced by C1, which made those comments stale.
- **C5** (`internal/fabriccli/doc.go`, new "Sandboxed-role guard" section): a package doc for the behaviour C1 adds, so forced by C1. Not unrequested scope.
- **C6** (`contracts/stencils/harden/harden-rubric-review.md:47`, one line): traces to G1. It tells the round agent, the actor G1 names, where the verbs now run. The line is accurate and drops or weakens no existing rule.
- **C7** (`internal/fabriccli/roleguard_test.go`: the new subtest, the `runUnderParentIn` / `fakeWorktree` / `sandboxHubDir` helpers, and the file header reworded in b6b1bf3): forced by C1, as the tests of the new behaviour.

**Decisions and the changes that build them:**
- **G1:** C1, C3, C6.
- **G2:** C1 (the ordering of its branches) and C2.
- **G3:** C1 and C3.
- Every decision is built.

**Findings:**

1. **Severity:** LOW. **Class:** scope. **Location:** `internal/fabriccli/roleguard.go:130`.
   **Summary:** the sandbox pass is open to every sandboxed task role, but the Goal licenses only "a hardener round's agent".
   **Evidence:**
   - The Goal (`_lym/discussion/decision-record.md:13`) names the actor as "a hardener round's agent".
   - `roleguard.go:130` applies `inOwnSandboxRoot(target, name.Slug)` to any parsed role except driver and orch, with no role check.
   - `roleguard_test.go:245` pins the pass for `webster`, which is not a hardener round role.
   - `_lym/landing/summary.md:19` records the widening as the writer's own design decision, not a recorded parent correction.

   **Why only LOW:** the effect is small. Only a worktree that holds a sandbox root is affected, and that is a hardener run's worktree. But the change does license more than the entry asks for.

   **Fix:** either limit the pass to the hardener round roles, or have the parent record the widening with `lym loom decision add`.

Tool calls: none. The inherited context already held the decision record, the diff pieces, the summary and the commit log.
