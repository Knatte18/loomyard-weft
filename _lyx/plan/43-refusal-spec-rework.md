# Card 43 — refusal-spec-rework

**Prosa:**
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `plugins/ly/skills/ly-drive/SKILL.md`

**Intent:**
Bring the refusal table and the two docs in line with cards 32–42, and fix PR #321 finding 4d, finding 7's spec lines, and the overview nit.
Each edited row's Way forward cell quotes the clause its message now carries, as the file's own rule requires.
Keep semantic line breaks; table cells stay on one line.

`refusal-spec.md`:

- Finding 7: reword every use of the two fabric side names.
  The way-forward definition's third item becomes "a plain `git` command in the task worktree, when no `lyx` verb covers it";
  the webster section's audit sentence says "a parent write into the task worktree's tracked content or under the run's `_lyx`";
  the `parent-write, elsewhere` row says "a git-ignored path in the task worktree outside `_lyx`".
  The file ends with no occurrence of either name.
- Card 36: the audit sentence adds a fabric reference whose command can change files to the correctness side;
  the `audit: fabric-reference` row splits into a correctness row (a mutating command: `lyx webster recover-batch NN` at record-batch; at run exit, the pending-finding way forward below) and a policy row (a read-only command: warn and record once).
- Card 37: the `rebaseline card set changed` row's trigger names a begun card whose content changed as well as a removed or regrouped one.
- Card 38 and 39: the `run-exit audit correctness` row's way forward names `lyx webster accept-audit` after the git revert, then re-stepping the Webster row;
  the `audit: parent-write, tracked or _lyx` row's run-exit clause says the same;
  a new `pending audit findings` row (run entry refuses while correctness findings from an earlier run exit are pending; correctness halt; revert or re-derive the named paths with git, then `lyx webster accept-audit`);
  the `no run in progress` row's trigger includes `accept-audit`.
- Finding 4d and card 40: every loom `approve` row says PR-Gate where it says Publish (`approve: not at PR-Gate`, and its trigger and way forward);
  the no-pull-request and not-open rows name `lyx loom goto --to Publish`, then `lyx loom step`;
  the HEAD-differs row names `git push` or `git pull`, then re-running `lyx loom approve`.
- Card 32: the `Loom-Preflight half-finished` row's way forward names seeding a new run, or `lyx loom goto --to Loom-Preflight` to accept the run as a deliberate re-entry, which then passes the check.
- Cards 33 and 34: the `current producer missing` and `bounce budget exhausted` rows name `lyx shed goto <run-id> --to <row>`;
  the `status file missing` row says the way forward is the recipe's own, as the message names it (`lyx loom start` for loom, `lyx batten run <slug>` for batten, `lyx shed seed` otherwise);
  two new goto rows: `goto on a running run` (correctness halt; `lyx shed pause <run-id>` then `lyx shed step <run-id>` leaves it paused, then re-run goto) and `goto target past the current row` (correctness halt; covers the awaiting narrowing; re-run goto with `--to` naming one of the rows the message lists).

`docs/overview.md`: the `internal/shedverbs/` line reads "the generic run/step/status/pause/goto cobra verb bodies".

`ly-drive/SKILL.md`'s "Ways forward" section: the two stops name `lyx shed goto <run-id> --to <producer>`, and one sentence says goto only moves a halted run back, to a row at or before its current row, so the driver never reports a forward goto as a way forward.
