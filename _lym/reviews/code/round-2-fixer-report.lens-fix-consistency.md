```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for agreement with its siblings (lens fix-consistency).

No list is asked for by this lens.

**1. MEDIUM — Go-filled way forward still tells Master to run `fix-verify` as a backgrounded Bash command**
- Changed text it depends on: the Master stencil's `fix-verify` step (`webster-template-master.md` step 2, now `{{template "engine-background-command"}}`).
- `internal/websterengine/verifygatereport.go:27` builds the cold way forward the gate sends Master. It says: "run `%s` as a backgrounded Bash command, as the gate-failure section of your prompt says; then end your turn."
- On Codex, the stencil's block now says to run it in the foreground through the shell tool and act on its output when it returns. The two instructions contradict.
- `verifygatereport_test.go:221` pins "backgrounded" in that text, so the twin was left unchanged and nothing flags it.
- The Go text would need wording that holds for both engines, such as "run `%s` as the gate-failure section of your prompt says". The decision to move the stencil passages (creel #1694) did not name this Go text.

**2. LOW — Landing summary still counts four allowlist entries**
- Changed text: `_lym/landing/summary.md`, the `engineterms_test.go` bullet.
- It says: "An allowlist with a reason per entry admits four Go literals that never reach a Codex session: shellguard's Claude row texts and heddle's help and HEDDLE.md place titles."
- F6's rekeying by declaration (`engineterms_test.go`'s `claudeLiteralAllowlist`) split overview.go into two entries (`placeTitle`, `placeReach`), so there are now five.
- The summary does not describe the new key shape either (file, declaration and term). Per the prose rule, the fix is to drop the tally and state the key shape.

**3. LOW — Pre-existing Go doc comments still describe Master's backgrounded calls as the stencil's wording**
- `internal/websterengine/doc.go:657`, `:888`, `:931` and `:1060-1062` state that Merriam runs these verbs backgrounded and acts on the completion notification. `runlevel.go:769` says the same.
- They now misdescribe the stencil on Codex, but they are Go comments and not agent-facing. Noted only; skip if the comments are out of scope.

**Checked, consistent**
- Block name `engine-background-command` matches the `background-command.md` file names.
- Embed vars, registry rows and both copies are in place.
- The Master stencil's in-place edits keep its line count, so `MerriamBase` is unaffected.
- The `agentmessaging_test.go` renames are consistent in all three places.
- The `engineterms_test.go` key doc and the synthetic rows agree.
- `webster-prefix-fork.md:25` ("woken by your completion notification") is a fork-side stencil outside this change set.

**Tool calls**
1. Grep for `backgrounded` and `completion notification` across the non-test Go and md files under `contracts`, `internal` and `cmd`, to find twins of the changed Master passages. My context held only the stencil, not the Go text or doc comments that echo it.
2. Grep for `fork-message` and `fork-wait` across md files, to see whether a doc enumerates the engine blocks and needs `background-command` added. My context did not show them. It turned up only stencils and research reports, none of which enumerate the blocks.
