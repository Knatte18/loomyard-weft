```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the review proposals the fixer carried out as written, looking for cases where the proposed change itself is wrong.

1. **MEDIUM — card 14, the phase-request bullet of `runRule`'s Intent.**
   - **Changed text:** "A phase request first calls `lock` with the phase, then calls `read`…"
   - **Summary:** the proposal (F4) says `lock` is called "with each phase before its first read". The card calls it on every phase request, so a rule that asks for the same phase twice takes a non-reentrant lock twice while the first release is still held. The second take deadlocks or fails.
   - **Concrete case:** a rule's under-lock phase read hits a transport failure. The card then records the failure and asks the rule again, and a rule that answers by asking for `PhaseUnderLock` again calls `lock(PhaseUnderLock)` a second time. Releases are held "until it returns, on every return path", so the verb's lock is still held at that point.
   - **Fix:** say `lock` is called once per phase, at that phase's first request. A later request of a phase already locked reads under the lock it holds. Add a `TestRunRule` row for a phase asked twice taking one lock.

2. **MEDIUM — card 3, the `TestHubPaths` bullet (F1's proposal).**
   - **Changed text:** "each want built from `lymdirs.BoltDirName` concatenated with literal segments, such as `"/h/" + lymdirs.BoltDirName + "/_lym/specs"`".
   - **Summary:** the moved functions build paths with `filepath.Join`. On Windows they return `\h\_bolt\_lym\specs`, so wants built from forward slashes fail there. The tests this replaces built their wants with `filepath.Join`, five calls in `junctionnames_test.go`.
   - **Why Windows matters:** lym runs there. `reedengine/doc.go` names psmux on Windows, and `livestate_doc_test.go` says the harness has no `runtime.GOOS` skip.
   - **Fix:** build the wants as `filepath.Join("/h", lymdirs.BoltDirName, "_lym", "specs")`, with the literal segments as separate `Join` arguments. This still avoids a literal `_bolt`. The `HubScratchDir` and `StencilsDir` sibling wants change the same way.

3. **LOW — cards 13 and 14, the timing of the allowlist doc comment.**
   - **Summary:** card 13 now adds the first temporary entry, `internal/fabriccore.Refusal.Error`, but `testOnlyCodeAllowed`'s doc comment only names that second kind of entry in card 14. For one card, the comment says every entry is a shared reference or helper, which the new entry is not.
   - **Evidence:** `cmd/lym/testonlycode_linux_test.go:41-46`.
   - **Note:** the parent's ruling names card 14 for the doc comment, so whether it moves to card 13 is the parent's call.

No other proposal broke on a case I could build. F6's `const StoreCode Store = fabriclayout.RepoCode` compiles because `RepoCode` is an untyped constant. F8's `ForgetOriginNamespaceForTest(dir string)` exists at `internal/fabricengine/export_test.go:661`. F5, F7, F9, F11, F13 and F14 hold.

Tool calls:
- One Bash call, which read three things my context lacked:
  - `gitexec.IsTransportFailure`'s body, to check how F5's transport row is classified.
  - Where `ForgetOriginNamespaceForTest` is declared and its signature, for F8.
  - Whether the repo runs CI on Windows (none found under `.github`) and the `filepath.Join` count in `junctionnames_test.go`, for finding 2.
