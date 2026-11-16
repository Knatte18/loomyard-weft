```yaml
lens: interactions
outcome: reported
```

Task: interactions-lens review of the Weaver decision record. I checked pairs of decisions that act on the same state, file, command or ordering.

**Pairs checked**

- D1 guard × D13 seam (begun batches, corrupt or missing state): held.
- D1 × D2 (`lym weaver reset` clears state, so the guard then sees no begun batches): held.
- D2 reset × D9 per-batch commit of `_lym/weaver/`: **broken** (F1).
- D2 run lock × D12 run lock: held.
- D2 Stuck reasons × D8 arbiter description: held.
- D3 batch numbering × D11 shared checks keyed by `batchIdentity`: **broken** (F2).
- D3 recorded partition × D7 re-entry checks after a plan edit by the arbiter or a reroute: **broken** (F6).
- D3 state-mutation lease × D13 renumber verbs: held.
- D3 card set fixed across attempts × D11 renames rewriting pending cards: held, since renames touch only pending cards.
- D4/D8 report and journal written into `_lym/weaver/` × D6 clean-tree check × D9 commit after the batch ends: **broken** (F3).
- D5 bound × D6 same-rung retry × D7 re-entry attempt: held, because D5 counts both.
- D5 `weaver.yaml` keys × D10 `verify_gate`: **broken**, wording (F8).
- D6 NEEDS_PARENT stop × D7 re-entry on the last rung: **broken** (F5).
- D6 stop-then-record-failed × D12 crash recovery: **broken** (F4).
- D6 machinery retry for fixers × D10 fixer × D12 crash and resume: **broken** (F7). The fixer has no recorded identity and no attach.
- D7 verify re-entry × D10 one fixer on the last rung: held.
- D8 journal append keyed by heading × D12 re-recording a report: held.
- D9 rework archive × D13 Describe's prior-generation readers: held.
- D11 `Geometry` run directory × D13 shared plan-conflict path: held.
- D12 attach sending decisions × D13 inline notices to live Weaver strands: **broken**, duplicates (F9).
- D12 `AttachGated`'s removal of superseded strands × D6 "respawn-eligible counts as a machinery loss": held.
- D13 seam falling back to `tapestry.implementer` × AC "a corrupt Webster state file under a Weaver run blocks neither": held.
- Acceptance criteria × decisions: the D12 criterion leaves out the fixer strand (part of F7). Every other criterion matches its decisions.
- Constraints × decisions (fabric-boundary × D9, shuttle-provider-seam × D6/D12, gate-slot × D10): held.

**Findings**

**F1 — MEDIUM, design — D2 × D9: the reset archives `_lym/weaver/` after the hard reset has deleted it.**
- D2 orders the steps as "moves the task branch back to the run's start commit … then archives `_lym/weaver/`".
- D9 commits `_lym/weaver/` onto the task branch after every batch, so the directory is tracked content the run's start commit does not hold.
- Moving the branch and tree back to the start commit therefore removes the directory from the worktree before the archive step reads it.
- Webster's own reset works because Webster commits its run directory only at Done, so the directory is untracked during the run.
- The decision is silent on this ordering, and the D2 acceptance criterion would not notice an empty archive.
- Fix: archive before moving the branch, or archive from the pre-reset HEAD.
- Evidence: decision-record.md:45, 121; Webster's commit-at-Done rule in D1's spec line, decision-record.md:29.

**F2 — MEDIUM, design — D3 × D11: "numbered after the done ones" conflicts with how the shared checks key batches.**
- The shared code identifies a batch by its first card's number (`batchIdentity`, `internal/websterengine/runlevel.go:982`).
- `unbegunCards` looks each partition batch up in `st.Batches` under that key (`recordbatch.go:114-124`), and `isLastBatch` compares the same key (`cardgate.go:66-73`).
- If D3 numbers batches by position, a begun batch keyed 2 that holds cards 3–4 resolves to `st.Batches[3]`. The check then reads it as unbegun, so the later-reference check runs over the batch's own cards. A positional number can also collide with another batch's first-card number.
- Fix: D3 keeps the shared numbering, the first card's number.
- Evidence: decision-record.md:54, 142.

**F3 — MEDIUM, design — D6 × D8/D9: the clean-tree check runs before the commit, while lym's own run-directory writes are still uncommitted.**
- D6 fails an attempt that "leaves the tree unclean", and that verdict decides whether the batch ends done or failed.
- D9 commits `_lym/weaver/` and the plan directory only "after each batch ends", which is after that verdict.
- Before the commit, the tree holds the agent's own report (D4: it is the strand's output file under Weaver's reports directory, D9), D8's journal append, and D11's rewrites of pending cards. None of these is ignored by git.
- Webster's `dirty` counts every porcelain line (`internal/websterengine/gitwrap.go:240-246`), and `.gitignore` does not exclude the run directories.
- As written, every attempt fails the clean check because of lym's own files. The record states neither an exclusion nor an order that commits first.
- Evidence: decision-record.md:64-65, 88, 108, 117, 121.

**F4 — LOW, design — D6 × D12: a crash between stopping a strand and recording the failure turns a model failure into a machinery loss.**
- D6 stops the failed or timed-out strand first, then records the attempt failed.
- If the process crashes between the two, state still shows the attempt in flight, and the strand is gone with no report.
- D12's attach then finds no attachable run: the stopped record is terminal, so `leftoverThenAgeVerdict` returns respawn-eligible (`internal/shuttleengine/attach.go:595-606`).
- D12 counts that as a machinery loss, so the batch gets a same-rung retry instead of climbing the ladder.
- The damage is bounded by "a second loss counts as failed".
- Fix: record the failure before stopping the strand.
- Evidence: decision-record.md:92, 156-158.

**F5 — LOW, design — D6 × D7: a NEEDS_PARENT stop, once answered, re-enters on the last rung.**
- D6 says a question "stops the row at once … without climbing".
- D7 gives every re-entry after a batch stop "one more attempt on the last rung".
- So once the parent answers, the batch jumps from rung 1 to the top rung, skipping the middle rungs. This contradicts D6's reason for not climbing on a question; the support log's Q5 says "a question is not a model failure".
- Evidence: decision-record.md:93, 100.

**F6 — LOW, design — D3 × D7: re-entry runs the checks against a partition that may no longer match the plan.**
- D7's re-entry runs the post-batch checks before any begin.
- D11 says those checks read the partition D3 recorded, and D3 replaces it only at a begin.
- An arbiter fix that edits or adds a card, or a reroute to Plan-Review between the stop and the re-entry, leaves a recorded partition naming different cards.
- Mapping a partition refuses on a plan card that no recorded batch holds (`partition.go:113-132`). That refusal names `lym webster rebaseline` and `restore-plan`, which are not Weaver's ways forward.
- Evidence: decision-record.md:54-56, 98.

**F7 — MEDIUM, design — D10 × D12: crash and resume cover batch attempts only, not the verify fixer strand.**
- D12 fixes a strand identity before the spawn, records it, and attaches on re-entry, but only for "every attempt of a batch".
- D10's fixer strand has no output file named and no identity recorded before its spawn. D6 even applies the same-rung machinery retry to fixers, which presumes losses are detected.
- After a crash mid-fixer, D7's verify re-entry, or a re-step, can start a second fixer beside the live one. That is the duplicate-agent race D12's own rationale names.
- The D12 acceptance criterion does not cover the fixer either.
- Evidence: decision-record.md:91, 129-130, 151-159, 226.

**F8 — LOW, consistency — D5 × D10: D5 lists `weaver.yaml`'s keys without `verify_gate`.**
- D5 lists `ladder` and `fixer_timeout_min` as what `weaver.yaml` holds; D10 adds `verify_gate`.
- Evidence: decision-record.md:71, 128.

**F9 — LOW, consistency — D12 × D13: an attached strand gets decisions it already received.**
- D13 delivers decisions inline to live Weaver strands while their run runs.
- D12's attach then sends "every decision entry recorded since its attempt began", which includes the ones already delivered.
- Fix: D12 sends only the entries recorded while the run was halted or the producer was down.
- Evidence: decision-record.md:157, 173.

**Tool calls**
1. grep for Webster's dirty and uncommitted-path helpers in `internal/websterengine/git.go`, `gitwrap.go` and `recordbatch.go`: the manifest did not cover how the clean-tree check treats the run directory (F3).
2. Read `internal/websterengine/gitwrap.go:225-264`: to see what `dirty` counts (F3).
3. grep `.gitignore` for the run directories: to check whether `_lym/webster` or `_lym/weaver` is ignored (F3).
