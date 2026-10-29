```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case, checking whether each step gets the input the step before it produces and whether the end state meets the decision's purpose.

**Decisions walked**
- D1, case: the prime has `pull_request: on` and the task worktree's `beam.yaml` says `off`. The value is read from the prime's anchor. Purpose met.
- D2, case: the parent is `required` and the field is `off`. The landing goes through a pull request and logs a Warn. Purpose met.
- D3, case: the row said yes and opened a pull request, then the parent changed to `off` before a Finalize reroute back into the row. The no closes the pull request and removes the record, and Finalize's resolver also says no, so it lands locally. Purpose met. One variant breaks: see F4.
- D4, case: a hub with a local origin and the entry field `on`. It halts. Purpose met, but the way-forward text is broken: see F5.
- D5, case: `pr_account` is set, the session has `GH_TOKEN`, and the token comes from `gh auth token --user`. Purpose met.
- D6, case: first entry, then a crash, then re-entry. The record is found and no second pull request opens. Purpose met.
- D7, case: the session is stopped at an approval whose head moved, then resumed. The resume's input, a session id, is produced by no earlier step. Purpose missed: F2.
- D8, case: the agent commits on a clean tree, the gate passes, the head is unchanged, the row pushes and records the head. Purpose met.
- D9, case: GitHub reports merged and the agent has an uncommitted edit. No bullet takes this state. Purpose missed: F3.
- D9, case: approved on the gated head with a clean tree. The row stops the session, re-reads, and returns Done. Purpose met.
- D10, case: a run waits at PR-Review and the session is idle past `notice_quiet_min`. Nothing marks the wait as live, so the quiet notice goes off. Purpose missed: F1.
- D11, case: a yes with a record, the parent synced, the fixer and gate green, then the GitHub merge and a fast-forward of the local parent pair. The weft side never lands. Purpose missed: F0.
- D11, case: the pull request was already merged on GitHub. Parent sync, the merge of the parent into the task branch, then the already-landed empty squash. The warp side is a no-op and the weft side merges. Purpose met.
- D12, case: `lym batten cancel` with an open pull request. The teardown closes it with a comment. Purpose met.
- D13, case: the pawl recipe, with Beam-Describe then PR-Review then Finalize. Wiring is consistent. Purpose met.

**Findings**

- **F0: BLOCKING, design, D11 "Otherwise" bullet.** A merge through GitHub replaces the local parent-side merge for the whole pair, but a pull request covers only the warp side.
  - After the GitHub merge, "fetches the parent and fast-forwards the local parent pair to its upstream" leaves the task's weft side unmerged, because origin's weft parent never received the task's records.
  - `Fabric.Merge` also never records the warp-to-weft link for the commit GitHub made.
  - So a landing through a pull request drops the run's records on the parent.
  - The record should route the post-merge step through Finalize's existing already-landed path, where the warp side is a no-op and the weft side really merges, rather than a plain fast-forward.
  - Evidence: `internal/landingshed/finalize.go:10-11` ("the remote service only ever sees one of the two sides, so the other side's branch was never in the pull request at all… the other genuinely merges") and `internal/landingshed/finalize.go:164-168`.

- **F1: BLOCKING, design, D10.** The run is not seen as waiting, so the quiet notice the operator ruled out goes off.
  - The marker "of a kind battenshed's quiet check already reads" has no fitting kind.
  - The check reads four things: a verify marker that is not paused, a parent-lock wait record, a child at `Beam-Await` by row name, and a shuttle wait marker that is not held.
  - The verify marker belongs to verifytree and lives only while a gate runs.
  - A parent-lock wait record contradicts D9's "takes no parent lock".
  - `Beam-Await` is matched by row name, and PR-Review is not that row.
  - A shuttle `wait.yaml` is written only inside a shuttle Run's own Wait, while `Runner.Open` "writes no run directory, no run record and no output file", and D7's session is not awaited.
  - In the idle-session case, `readAgentActivity` returns `waitLive=false`.
  - Settling this needs a new kind, for example a status running at PR-Review read the way `childAtBeamAwait` reads Beam-Await, and the record leaves that fact for Plan.
  - Evidence: `internal/battencli/wire.go:590-645`, `internal/battencli/wire.go:648-666`, `internal/shuttleengine/open.go:1` and `:43`, `internal/shuttleengine/waitmark.go:1-3`.

- **F2: MEDIUM, design, D7 and D9.** "A stopped session resumes it by its session id" has no source for the id.
  - `Runner.Open` writes no run record.
  - The D6 record holds only the number, URL, base and gated head.
  - `StopStrand` with no run record only removes the strand.
  - So after D9's stop-then-resume at an approval whose head moved, or after any re-entry, nothing gives the row the session id. The conversation is lost, and the call starts a fresh session instead.
  - The record must say where the session id is kept, for example in the D6 record.
  - Evidence: `internal/shuttleengine/open.go:1` and `:37-43` (`OpenResult.SessionID`, "writes no run record"), `internal/shuttleengine/stop.go:51-60`.

- **F3: MEDIUM, design, D9 outcome list.** The list does not cover "merged, merged head equal to the task head, tree dirty".
  - The first bullet needs a clean tree, the second needs commits the merged head lacks, and the "otherwise" bullet covers only open pull requests.
  - With the agent mid-edit at merge time, the row has no rule.
  - The loop either spins with nothing that ends it, or Plan invents the outcome.
  - It needs a stated outcome, for example the stop-and-re-read sequence used for an approval.
  - Evidence: `_lym/discussion/decision-record.md:151-155`.

- **F4: LOW, consistency, D11's first bullet against D12.** D12 says "a landed run's pull request is merged and left as it is".
  - In D11's case, the resolver says no at Finalize after PR-Review said yes, for example because the parent's mode changed during the wait.
  - That run lands locally with its pull request still open, and the teardown then closes it with a comment naming the teardown's cause.
  - So a landed run's pull request is closed as though it were abandoned.
  - Evidence: `_lym/discussion/decision-record.md:176` and `:192-193`.

- **F5: LOW, consistency, D4's way forward.** The text reads "…or … `lym batten cancel &lt;slug&gt;` and a new run with the field changed; then `lym loom resume`".
  - The trailing resume also attaches to the cancel branch.
  - After a cancel there is no run left to resume, so the halt text gives an unreachable step.
  - Evidence: `_lym/discussion/decision-record.md:83`.

**Tool calls**
1. `finalize.go:150-175` and shuttle resume and stop symbols: needed the already-landed path and how a session id is kept, which the context lacked.
2. The gateslot template and landingshed gate keys: to confirm that the `finalize` and `diff` scope semantics D8 and D11 rely on exist.
3. `OpenSpec` and `stop.go` `StopStrand`: to check whether an opened session leaves a record or session id to resume from.
4. Pawl recipe rows: to walk D13 and D6's description input for the pawl recipe.
5. loomcli and pawl `goto` references: to check that D11's `lym loom goto PR-Review` way forward is reachable on a pawl run. Inconclusive, no finding.
6. The earlier `readAgentActivity` read was already in context and was not repeated, so only 5 calls were made.
