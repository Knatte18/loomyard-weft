```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attack every claim and bound the pr-review-row decision record states.

**Claims attacked**
- D1, "a task changing its own `_lym/config/beam.yaml` reaches nothing; Finalize never carries per-worktree config files to the parent": **held.** The config files live on the weft side, and `ReadConfigChanges` reads them on the weft branch (internal/fabricengine/configchanges.go:16). A GitHub pull request covers only the warp side, so a GitHub merge cannot carry them either.
- D2, "the board lock freezes the `pull_request` field while the run holds the entry": **held.** The run lock refuses any net change to a field other than status, apart from its named exceptions (run_settings, owner, dates) (internal/boardengine/doc.go:204). A new field falls under the refusal unless someone adds it to those exceptions.
- D4, "an explicit demand never lands without a pull request": **held** on the warp side. See D11 below for the other side.
- D6, "a re-entry opens no second pull request": **held.** A leftover open pull request from a cancelled earlier run, whose D12 close failed, is adopted. Its approvals of old heads do not count under D9's head-equality rule.
- D7, "the session never pushes, never approves, merges or closes the pull request and never calls GitHub": **broken.** See F2.
- D7, "a stopped session is resumed by its session id": **broken** at LOW. See F5.
- D8 bound, "an ungated agent commit lands nothing until the row has gated it, because D9's Done and D11 read only the record's gated head": **broken.** See F1.
- D8, "a verdict counts only on an unchanged head and a clean tree": **broken** at LOW. See F4.
- D9, "wakes at least a second apart, GitHub polled at most once a minute": **held.**
- D9, "a pause ends the call with a transient mark": not attacked past the record; this is outside what my lens can build a counterexample for.
- D11, "goes on as after today's parent-side merge" after a GitHub merge: **broken.** See F3.
- D11, the halt on "a merge GitHub still refuses", with "approve again or settle the check" as the way forward: **broken** at MEDIUM. See F6.
- D11 bound, "what reaches the parent after the approval is the merge of the parent and the fixer's commits": **held** for lym's own merge path. It does not hold for the D9 merged path; see F1.
- D12, "every teardown closes an open pull request": **held**, given the stated Warn-and-continue on failure.
- D5, "every GitHub call of D3 to D12 goes through the `pr_account` client": **held** as stated.

**Findings**

**F1. BLOCKING, design, D8 bound and D9 "merged" ending.**
D8's bound is breakable through D9's merged ending.
- The detached records push carries an ungated agent commit to the pull-request branch, as D8 itself admits.
- Anyone, including the agent itself (F2), can then press merge on GitHub.
- D9's merged case is Done when "the merged head equals the task head and the tree is clean". It never compares against the record's gated head.
- Finalize then takes the already-landed path, so an ungated head is on the parent with no gate.

So the claim "D9's Done and D11 read only the record's gated head" is false for the merged case.
Evidence: decision-record.md lines 138 and 151; the already-landed idempotence in internal/landingshed/finalize.go:164-168.

**F2. BLOCKING, design, D7.**
The limit "never pushes, never approves, merges or closes the pull request and never calls GitHub" exists only as an instruction in the stencil. The Decision names no setting or guard that enforces it.
- The session runs as an interactive agent in the task worktree, with the operator's environment, where `git push` and `gh` both work.
- The shipped claudeengine template denies only the Agent tool and AskUserQuestion (internal/shuttleengine/claudeengine/template.yaml:2-3). I found no deny for `git push` or `gh`.
- So the agent can `git push` an ungated head, or `gh pr merge` the pull request, which reaches the parent through F1's path.

The attack-surface rule requires the Decision to state what bounds the session; it states none.
Evidence: decision-record.md line 118; template.yaml:2-3.

**F3. BLOCKING, design, D11's GitHub-merge bullet.**
"In place of the local parent-side merge, merge the pull request through GitHub, then fetch the parent and fast-forward the local parent pair to its upstream" covers only one side.
- A GitHub pull request covers the warp side only. Finalize's own header says the remote service sees one of the two sides, and the other side's branch was never in the pull request (internal/landingshed/finalize.go:10-11).
- The weft parent's upstream does not hold the task's records, so a fast-forward of the pair lands nothing on the weft side.
- The weft-to-warp correspondence for the commit GitHub creates is never recorded. Fabric anchors weft to warp through `Warp-SHA` trailers (internal/fabricengine/doc.go:12-13).

The run is marked done with its records never merged into the parent.
Evidence: decision-record.md lines 179-181; finalize.go:10-11; fabricengine/doc.go:12-13.

**F4. LOW, design, D8 verdict check.**
The check "head unchanged and tree clean after the gate as before it" misses an edit that is undone mid-gate.
- The agent works in the same worktree the gate verifies, with no lock between them.
- If it edits a file and reverts it while the gate runs, the head is unchanged and the tree is clean after.
- The verdict then counts for a tree the gate did not fully read.

Evidence: decision-record.md line 134.

**F5. LOW, design, D7 resume.**
"A stopped session is resumed by its session id": `Runner.Open` writes no run record and returns the session id only to its caller (internal/shuttleengine/open.go:1, 26-29, 100).
The record never says where the row stores the id for a later call, or after a crash. The D6 record's listed fields (line 106) do not include it, so a re-entry after a crash has no id to resume.
Evidence: open.go:1, 26-29, 100; decision-record.md lines 106 and 122.

**F6. MEDIUM, design, D11 refusal halt.**
The halt's way forward ("approve the pull request again or settle the check, then `lym loom resume`") does not cover every refusal it catches.
- A repository that disallows the configured method, such as `squash` with squash merging turned off, makes GitHub refuse every merge.
- Neither re-approval nor a check fixes that, so the resume halts again, which breaks `PATTERN-refusal-way-forward`.
- The halt needs a way forward per refusal class, or the method checked when the pull request is opened.

Evidence: decision-record.md lines 179 and 182; PATTERN-refusal-way-forward.

**Tool calls**
1. finalize.go:150-175, the open.go session-id lines and a grep for config in finalize: to check D7's resume claim and Finalize's already-landed path one hop past the manifest.
2. fabricengine doc lines on push and detached push: to check D8's claim about the detached push and the two-sided topology.
3. finalize.go:440-470: to check D1's "Finalize never carries per-worktree config" claim at its source.
4. A grep for `ReadConfigChanges` and its callers: to learn which side the per-worktree config lives on, which decides whether a GitHub merge can carry it.
5. A grep for deny rules in contracts and shuttleengine: to check whether D7's "never pushes, never calls GitHub" is enforced anywhere.
6. A grep of the claudeengine template and settings golden file for push and gh denies: same purpose as call 5. The output was too large, and only the template's first lines were readable.
