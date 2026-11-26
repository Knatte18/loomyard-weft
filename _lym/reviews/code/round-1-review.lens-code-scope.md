```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I mapped each change in the diff to a goal point of the board entry and checked for unrequested or missing scope.

**Decisions.** The answer key is the decision record alone: the board entry under `## Goal`, with no `Added after Discussion` entries.
- **G1:** the refusal names the anchored root and says that a previous `cd` likely moved the shell. The alternative is to accept calls from any directory inside the worktree.
- **G2:** the agent guidance mentions that `cd` persists, and that absolute paths or `git -C` avoid it.

**Per change**
- **C1:** `internal/lymcwd/anchor.go`, `checkCwdAnchorGate`, error text and doc comment (lines 108–118).
  - Traces to G1, taking its first remedy.
  - The message names the anchored directory and says that "an earlier cd likely moved it".
  - It adds a `way forward:` clause, which belongs to the same purpose.
- **C2:** `internal/lymcwd/gate_test.go`, the `wantMessage` field, its row and its assertion (lines 33–99). Forced by C1: it is the test that reaches the changed refusal.
- **C3:** `internal/heddleengine/shipped.json`, the `lym-shell-directory` entry (lines 19–26).
  - Traces to G2.
  - It says the shell keeps a `cd` for later commands and names an absolute path and `git -C &lt;dir&gt;`.
  - It also names `env -C &lt;dir&gt;`, which the entry does not list. I judge it a third instance of the same advice to avoid `cd`, within G2's purpose, not a separate change.
- **C4:** `internal/heddleengine/store_test.go`, the `ShippedEntriesArePinned` row (line 188). Forced by C3: the test fails on its length check (`len(ordered) != len(tests)`) without a row for the new entry.

**Per decision**
- **G1:** built by C1, with C2 forced by it. The diff takes the first of the two remedies the entry offers ("or"), so not building the subdirectory-acceptance alternative is no gap.
- **G2:** built by C3, with C4 forced by it.

**Findings:** none. Every change traces to a goal point or is forced by one, and every goal point is built.

**Rejected candidate:** `env -C` in C3 as unrequested scope. It is the same guidance G2 asks for, avoiding `cd` through a directory-taking form, not a change with its own purpose. Whether `env -C` is usable on every supported platform is a correctness question for the other lenses and the handler, not a scope question.

**Tool calls:** none; every file this lens needed was already in the inherited context.
