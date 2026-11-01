```yaml
lens: fix-way-forward
outcome: reported
```

Fork lens fix-way-forward: checking that every refusal, next step, row and test the fix introduced can be carried out from the state it names and makes progress.

Findings:

1. **MEDIUM.** D6, "While any front task stands open in the subject's worktree … the check files none", together with D8's start-hold lift "or in any other status, such as the draft status".
   - **Summary:** "open" is never defined. A front entry left as a draft by a hand `lym batten cancel <slug>` without `--remove` lifts the start hold, but it still carries `front`. Every later failed check therefore files nothing, and D5's green end removes the fix task only "if one stands halted", so it never removes a draft one.
   - **Evidence:** cancelcmd.go's help says "The pair is kept by default, and \"lym batten cancel <slug> --remove\" must run before the slug runs again". `clearRunClaim` writes `StatusDraft`. The F3 wedge comes back on the filing side through a hand cancel. The way forward is to define "open" with D8's status rule, or to have the green end remove a draft fix task as well as a halted one.

2. **MEDIUM.** D6's failed-notice clause, "the failed notice names that task instead".
   - **Summary:** When the open front task is not the adopt fix task (a note `parent-merge-task` row), its landing runs only its own `on_land` action. Nothing re-runs the adopt check, so the subject's runs stay held, and the notice names the task without naming the way forward.
   - **Evidence:** D8 says "the table's one row now is `adopt-check` … Note `parent-merge-task` adds its own row". D5 starts a check only from adopt or from `adopt-check`. The notice should also name a re-run of `lym adopt <subject>` once that task lands.

3. **MEDIUM.** D8, "`fronttask` removes a front task by `lym batten cancel <slug> --remove`, then the board entry's removal, and drops its record first", and D5's green-end removal.
   - **Summary:** The removal can fail after the record is gone, and nothing reports it or leads anywhere. The front entry then stays halted, so the start hold keeps standing. The holds are already lifted, so a re-run of `lym adopt` starts no check (D5 starts one only on an owner write or a standing hold).
   - **Evidence:** cancel refuses "while a hub pause covers the prime, … when origin stopped the entry, when another cancel of the slug is under way". With origin unreachable "it only ends the run's processes and strands, leaves its marker". D5's failure line covers only "A start or resume that fails at the green end".

4. **MEDIUM.** D7's new `halted` state, "naming the way forward its stop gives", and the D7 acceptance item for "a fix task stopped on a machine failure, naming its way forward".
   - **Summary:** The darn stop's own way forward is `lym loom decision add` and then `lym loom resume`. Resuming the writer on a machine failure makes no progress. D6 says the way forward for a halted fix task is to settle the machine and re-run `lym adopt <subject>`, but neither the `halted` line nor D6's body bullet makes the stop name that.
   - **Evidence:** `_bolt/_lym/stencils/darn/darn-template-write.md:50` reads "as the way forward `lym loom decision add` and then `lym loom resume`". D6 says "the operator, once the machine is settled, re-runs `lym adopt <subject>`". D6's body bullet says only "with the exact steps the operator must take".

5. **LOW.** D8 bound, "A filing that dies between the board write and the record leaves a front task whose landing runs no action; the operator's re-run of `lym adopt` starts the check."
   - **Summary:** The re-run works, because the holds are still standing. But nothing tells the operator to do it: the landed fix posts nothing and the subject stays held with no notice.
   - **Evidence:** D8 says "only a caller of its closed set calls it". No notice is named for this case.

6. **LOW.** D8, "the refusal names that only lym files a front task", and the reworded constraint "`PATTERN-refusal-way-forward`: `boardengine`'s refusal of `front` and `on_land` names its way forward".
   - **Summary:** "Only lym files a front task" is a reason, not a step. The refusal names no action for the caller, for example dropping `front` and `on_land` from the payload. The fix carried this wording into `upsert-batch` and `merge` while the constraint claims it names a way forward.
   - **Evidence:** the D8 bullet text against the constraint line as quoted.

Calls made:
- I read the `lym batten cancel` help and `refuseOwnPair` in `internal/battencli/cancelcmd.go` and `internal/battencli/refusal.go`, because my context had only grep lines, not cancel's refusals.
- I searched the worktree's `_bolt/_lym/stencils` for the darn writer stencil's stop text. It found nothing there, so I looked for the stencil elsewhere.
- I read the stop lines of `/home/knatte/Code/loomyard-LYMHUB/_bolt/_lym/stencils/darn/darn-template-write.md` to get the darn stop's way forward, which my context lacked.
