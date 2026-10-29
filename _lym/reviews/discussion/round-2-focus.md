---
round: 2
exclude_lenses: []
focus:
  - "In `_lym/discussion/decision-record.md` D10, check that the wait marker the PR-Review row keeps is a concrete mechanism with a stated lifetime, one of the live-wait kinds `readAgentActivity` in `internal/battencli/wire.go` counts or a stated new one, and that no part of the record leaves the choice to Plan."
  - "In D8's Bound and D9's merged-on-GitHub ending, check that both name the same head: whether a pull request merged on GitHub must hold the gated head, what happens when it holds an ungated one, and that Finalize's already-landed path in `internal/landingshed/finalize.go` cannot land an ungated head."
  - "In D7, check that every escape the session has in the task worktree (`git push`, `gh`) is bounded by something the Decision names and that is enforced outside the prompt, or that the residual is stated with what catches it."
  - "In D6 and D7, check that the session's strand GUID and session id are stored where D7's stop and resume read them, and that a shuttle verb to resume an opened session is stated, given `Runner.Open` in `internal/shuttleengine/open.go` returns them only to its caller."
  - "In D11's GitHub-merge bullet, check what carries the records-side work `Fabric.Merge` does under the weft write lock (`internal/fabricengine/merge.go`), and in D11's refused-merge halt check that each refusal class has a way forward that reaches a landing."
  - "In D9, walk the merged and approved states with a live session, a closed session and a dirty tree, and check each combination matches one bullet and agrees with D7's statement on when the session is started or resumed."
  - "In D12 and D3, check every teardown path, including a task worktree already gone, against the claim of what happens to an open pull request."
---
