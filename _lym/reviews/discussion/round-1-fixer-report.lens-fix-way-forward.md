```yaml
lens: fix-way-forward
outcome: reported
```

Task: check, under the fix-way-forward lens, that every refusal, next step, row and criterion the fix added can be reached and makes progress.

**Ways forward the fix introduced**
1. D3: refusal of a current name that is another machine's former name, with the way forward `lym machine name <new>`.
2. D3: refusal of a current name that another machine holds as its current name, with the same way forward.
3. D1: `PushOwed` refusal text for a push owed on a held teardown lock.
4. D2: hand-over mark, and the dobby's removal of a pair a mark names (D7's new bullet).
5. D5: upgrade notice adds `lym tender resume <slug>` after the green check.
6. D5: upgrade writes an `adopt-fix` hold from the local tries file.
7. D6 step 1: a re-run under this machine's own hold keeps the hold and goes on from step 2.
8. D6: the shell-guard row's steer and notice for `--take-hold`.
9. D2: a leased code-tip push, then a reset with a lease.
10. D4 step 4: archive tags and a notice.
11. D5: an owed push gets no `bolt-hold` entry.
12. D5: a local parent-side merge counts as past the merge under a hold.
13. D10: launcher at the first owned dobby start.

**Findings**

- **MEDIUM. D3, the new refusals with `lym machine name <new>` as their way forward: following it leads into another refusal or into a split identity.**
  - `lym machine name` puts the old name among this machine's former names (D3 criterion "puts the old name among the registry's former names").
  - Second refusal (current name equals another machine's current name X): after the rename, this machine's former names include X, which is still the other machine's current name. The existing rule "A registration whose former name is another registered machine's current name is refused" then refuses again. The operator needs `--forget-former X` as well, and the refusal doesn't say so.
  - First refusal (X is another machine's former name): after the rename, X becomes a former name of both machines. "A former name of a registered machine reads as that machine" then maps X to two machines, the clash the refusal was meant to prevent.
  - Fix needed: the way forward should name `lym machine name <new>` followed by `--forget-former <name>`, or the rename should not carry a refused name into the former names.

- **MEDIUM. D7's new bullet "The dobby also removes … the local pair of a run a hand-over mark of this machine names (D2), under a hold too: the run is another machine's": the removal doesn't check the owner, so it tears down this machine's own pair after a re-adopt.**
  - The mark persists, and the bullet asserts the run is another machine's without checking.
  - When this machine later adopts the run back, D6 step 2 "adds the pair from origin when this machine has none". The dobby's next pass then removes that pair, because a mark still names the slug.
  - This undoes the re-adopt with no way out. The removal should require that the current owner is another machine, or the mark should be dropped when this machine writes or reads a takeover naming itself.

- **MEDIUM. D6 step 1: "A hold this machine wrote stays as it stands … the adopt goes on from step 2" carries no subject condition.**
  - `--take-hold` refuses a hold naming other subjects; a plain re-run does not.
  - Example: under this machine's `adopt-fix` hold for subject A, `lym adopt B` keeps A's hold, and B's green check in step 8 "clears the hold".
  - That releases the hub while A's fix task is still open, against D5's bound that a hold stands until the fix task's re-check is green.
  - The plain re-run needs the same subject refusal (naming the hold's subjects), or the rule should be limited to a re-run of the hold's own subjects.

- **MEDIUM. D1: "Its refusal names the owed push and its last error … in place of today's text that origin is unreachable" rewords the reason but not the way forward.**
  - Today's way forward is "run %s again once origin answers" (`internal/runfsm/act.go:116`).
  - For a push owed on a held teardown lock, origin already answers, so re-running at once refuses again until the lock releases. The named step makes no progress.
  - `PATTERN-refusal-way-forward` needs a way forward for the lock case, such as re-running once the teardown lock is released or the owed push lands.

- **LOW. D6: "The row's steer and notice say only that an adopt's takeover of another machine's hold is the operator's".**
  - The steer is the deny reason shown to the agent, and `PATTERN-refusal-way-forward` requires every refusal to name its way forward.
  - The text should name the agent's next step, such as asking its parent or the operator, still without spelling the flag.

No further tool calls made.
