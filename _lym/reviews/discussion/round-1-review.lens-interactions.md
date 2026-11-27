```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of decisions that act on the same state.

**Pairs checked**
- D1 × D5, the board `done` push and its order after the parent push: broken (F1)
- D5 × D6, a local landing with origin unreachable against the owner check's wait: broken (F2)
- D5 × D5/D3, a second local landing on a parent whose first landing push is still owed: broken (F3)
- D5 × D1/PATTERN-pair-remote, the records commit through the pair's pusher against the parent-first order: broken (F4)
- D3 × D4/D7, a refused target leaving the ledger against the hub clear time and the `sync:` field: broken (F5)
- D3 × D5/D13, "owes a landing push" read through the ledger after a refusal: broken (F6)
- D3 × D6 and the Auto-mode assumption, a waiting run with an empty ledger: broken (F7)
- D11 × D13, Worktree-Teardown with pushes off: broken (F8)
- D1 × D8, the catch-up's own synchronous push: broken (F9)
- D2 × D3, the count of config keys: broken (F10)
- D2 × D5/D13, the push-now mark against the interval and the ledger: held
- D6 × D7, a refusal leaving the network wait: held
- D6 × D5, a pull-request landing stopping as `git-transport` before the parent lock: held
- D6 × D3, the all-clear waiting for both an empty ledger and no waiting run: held
- D7 × D1, the prime code branch fast-forward only, outside the reset rule: held
- D12 × D13/D3, the #218 hold lifted while the pair stands: held
- D9 × D5, the resume path past the merge: held
- D3 × D13, the entry dropped on removal while the pair stands owed: held

**Findings**

F1 — BLOCKING, design, decision-record.md D5 (with D1)
- **Summary:** Finalize writes the board `done` with `SetStatusPushed`, a synchronous push. No decision says what happens to that push.
- **Two outcomes, both broken:** if it stays synchronous, an unreachable origin fails the landing, against D5's "never fails". If it goes to D1's board target, the board pusher pushes every owed board commit on its own, so `done` can reach origin before the parent push. That breaks D5's promise that the landing target pushes `done` only after the parent landed.
- **Missing pieces:** D1's list of pushes that stay synchronous does not name it. D5 never says how a board commit is held back from the board target.
- **Evidence:** internal/loomcli/landingdeps.go:238-244; internal/boardengine/board.go:216-217; internal/landingshed/finalize.go:996.

F2 — BLOCKING, design, decision-record.md D5/D6 and their Acceptance criteria
- **Summary:** With origin unreachable, a local landing never returns done. It waits as a `git-transport` stop.
- **Why:** Before the merge, `ownerChecks` runs the run-scope reconcile against origin. D6 turns an unreachable origin there into a `git-transport` stop that the driver re-steps without a cap. `handedOverStop` also reads origin's owner before the push, and `syncParent` reads the parent tips with a fetch.
- **Effect:** the D5 criterion ("returns done while origin is unreachable") contradicts the D6 criterion (the owner check on an unreachable origin is a `git-transport` stop), and the Goal's "never fails" holds only as "waits".
- **Evidence:** internal/landingshed/finalize_owner.go:47-55, 120-128; internal/landingshed/finalize.go:439, 612, 957.

F3 — BLOCKING, design, decision-record.md D5
- **Summary:** After a local landing whose push is owed, the local parent is ahead of origin, and D5 does not cover that case.
- **Why:** D5 covers only a parent that lags origin (fast-forward) or has diverged (refusal). `syncParent` halts a parent that is ahead: "the local parent branch is ahead of its upstream; way forward: push or reconcile".
- **Effect:** every later local landing on that parent halts until the first push lands, against D5's "other runs keep committing" and the operator's intent.
- **Evidence:** internal/landingshed/finalize.go:616-622.

F4 — MEDIUM, design, decision-record.md D5
- **Summary:** D5 sends Finalize's records commit through the pair's pusher, which pushes on its own schedule. Origin's run record can then read Finalize done while origin's parent lacks the work.
- **Why:** today `pushParent` pushes the parent, then the records, then the board `done`, in that order. D5 keeps the ordering promise for the board `done` alone.
- **Who misreads it:** `runReader` reads origin's child status (`childOnOrigin`) and origin's parent (`originParentHolds`) for another machine or a reader whose pair is absent.
- **Evidence:** internal/landingshed/finalize.go:933-996; internal/pairteardown/runread.go:129, 222-224, 269-285.

F5 — MEDIUM, design, decision-record.md D3/D4 (with D7)
- **Summary:** A refused target leaves the ledger, so the hub file can record "no owed target", and the all-clear can be posted, while that branch is still ahead of origin.
- **Why:** the dobby's ahead check writes an entry only "when it starts a pusher". `Start` refuses while a refused-push record stands.
- **Effect:** `sync:` reads `0m` while work is only local, which defeats D4's stated purpose. The failed-push notice is the only signal left.
- **Evidence:** internal/runpusher/start.go:122-125; decision-record.md D3 "A target whose push is refused leaves the ledger", D4 "minutes count from the time the hub last had no owed target".

F6 — MEDIUM, design, decision-record.md D5/D13 (with D3)
- **Summary:** Once a landing push is refused, its entry leaves the ledger and "owes a landing push" reads false. Three reads that depend on it then go wrong:
  - the run reader's past-the-merge read switches to origin's parent, which lacks the work, so the run reads as not past its merge;
  - D13's landed-pair teardown no longer waits on the parent;
  - D5's pull-request check sees no owed push. It should see a blocked halt, and D5 says a refused one is exactly that.
- **Fix needed:** D5 and D13 must say whether "owes" means "in the ledger" or "ahead of origin", and how a refusal counts.
- **Evidence:** decision-record.md D3, D5, D13; internal/pairteardown/runread.go:273-280.

F7 — MEDIUM, consistency, decision-record.md D3/D6 and Auto-mode assumptions
- **Summary:** D3 posts its summary only once some ledger target has stayed owed past the threshold.
- **Effect:** a run that D6 holds waiting on a GitHub API outage, with no push owed, never triggers a summary. D6 says the waiting run "shows only in D3's summary", and its bound relies on "D3's summary makes a long wait visible", so such a wait is never reported.
- **Assumption not met:** the Auto-mode assumption says the threshold "also governs when a waiting run enters the summary", but D3 gives a waiting run no trigger of its own.
- **Evidence:** decision-record.md D3 "Once some target has stayed owed…", D6 bound, Auto-mode assumptions bullet 3.

F8 — MEDIUM, consistency, decision-record.md D11/D13
- **Summary:** With git or pushes off, origin's tracking refs never hold the tips. Under D13, Worktree-Teardown then always leaves the pair standing, and the dobby never removes it.
- **Contradiction:** the D11 criterion says that, with pushes off, Worktree-Teardown pushes nothing and removes the pair once. The PATTERN-pair-remote background says the same: it "removes the pair without that read". D13 makes no exception for pushes off.
- **Evidence:** pattern/PATTERN-pair-remote.md:99-101; decision-record.md D11, D13.

F9 — LOW, consistency, decision-record.md D1 (with D8)
- **Summary:** D1 claims every lym push either moves to the pusher or is named as keeping its own push. The dobby catch-up's `pushOwedForCatchUp`, a synchronous push with a 30 s lock wait, is neither.
- **Why it matters:** D8 keeps that push and adds a hand-off on an owed failure, so the push stays synchronous while D1 does not name it.
- **Evidence:** internal/battencli/catchup.go:263-276; decision-record.md D1 last paragraph.

F10 — NIT, consistency, decision-record.md D2/D3
- **Summary:** D2 says "Two keys in `internal/fabricengine/template.yaml`" and "the keys are new, so no reinterpretation ships". D3 then adds a third new key, `fabric.push_notice_after_min`, without saying it goes in that template.
- **Evidence:** decision-record.md D2, D3.

**Tool calls**
- `grep MarkTaskDone: internal/loomcli/landingdeps.go`: to learn how Finalize's board `done` is written and pushed. The manifest held `markTaskDone` but not its wiring.
