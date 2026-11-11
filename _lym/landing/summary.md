# Add `lym loom decision replace` and cut the history check at it

A run's decision record past Discussion approval could only grow through `lym loom decision add`.
A changed or withdrawn call stayed in the record as a stale entry, and the board copy of a raddle run's Sketch went stale.
This change adds a whole-record `replace` verb, bounds the dropped-entry check at the newest replace, and keeps every stored record current.

## What changes

- **`lym loom decision replace [<slug>] --from <file> --title <text> --base-sha256 <hex> --by parent|operator|arbiter`**
  in `internal/loomcli/decision.go`.
  - It takes `add`'s caller guards, so only the run's parent, its arbiter, the run's sampler strand with `--by operator`, or the operator's unnamed shell replaces.
  - It takes `add`'s refusals too: Discussion-Write running, a perch raddle run at its Raddle-File row, and a Record-Write run with an uncommitted or diverged record.
  - It is also refused on a run with a Discussion approval row that has not passed it, and on a run between its plan's approval and its recipe's last Webster row.
    Both refusals name `lym loom decision add` as the way forward.
    The Webster span is read through the new `loomrecipe.InImplementerSpan`.
  - It refuses unless the committed record's sha256 equals `--base-sha256` and the working copy equals the committed one.
    The refusal names no hash.
  - The file must pass `discussionparser.Validate` and `CheckDecisionIDs`, keep every `lym:carry-over` block byte for byte, and keep each `### Added after Discussion` entry byte for byte.
    It may drop added entries and appended `## PATTERN changes` bullets, and may rewrite a fenced board body in `## Goal`.
    The new `discussionparser` checks are in `replacecheck.go`.
  - It commits through `fabricengine` with a `Lym-Record-Replaced: <by> <path> <sha256>` trailer.
    A replace whose bytes HEAD already holds lands an empty trailer commit through the new `fabricengine.CommitEmptyWithTrailers`
    and `gitrepo.Repo.CommitPathAllowEmpty`, which leaves other staged content staged.
  - A commit error is read against HEAD:
    with the written bytes and the trailer at HEAD the call succeeds and reports `commit_error`;
    with the bytes alone it lands the trailer commit;
    without the bytes it restores the prior record and refuses with a way forward that names `replace`.
  - It posts each live strand a creel notice that opens with `Parent decision record replaced with lym loom decision replace`.
- **A per-run decision lock**, formed through `internal/lock`, is held by `add`, `replace`, `store` and the Bouncer carry-over writer `writeCarryOver`.
  A second writer waits, so the base-hash check and its write are atomic against a concurrent `add` or carry-over write.
- **The history check is cut at the newest replace.**
  `recordHistory` in `internal/loomcli/projectrecord.go` finds the newest `Lym-Record-Replaced` trailer that names the record's path.
  It returns the versions from the newest version whose bytes hash to the trailer's sha256 and that sits at or before the trailer commit.
  Order is first-parent position on the records branch, never committer time.
  With no matching trailer, or a sha256 that matches no version, the history is read whole as before.
  `fabricengine` gains two general seams for this: a first-parent trailer-key commit list and an anchored file's first-parent history with commit SHAs and positions.
  The trailer key and the cut stay in `internal/loomcli`.
- **`add`, `replace` and `store` refresh the board copy of every stored record.**
  - The copy runs on a run past its Discussion approval, and on a run whose recipe has no approval row when the entry carries a stored record.
    That covers a raddle run of a promoted Sketch and a darn part filed by raddle.
  - A beam's project record is excluded, and a run sent back into its Discussion segment copies nothing.
  - Once a raddle run's filing is done, `add`, `replace` and `store` are refused in every mode.
    The filing counts as done by `raddleRunFiling`, by the entry no longer being a task, or by `Board.HoldsRaddleFiling`.
    The way forward names `add` or `replace` on the filed part, or `lym board upsert --record-file` on the family's project.
    A filing that saved `board.json` but wrote no done marker names `lym loom resume`.
- **Specs and directive.**
  `contracts/specs/loom-discussion-spec.md`, `contracts/stencils/parent/parent-directive-parent.md` and `docs/overview.md` say when to `replace` and when to `add`.
  They also say that the dropped-entry check reads back to the newest replace,
  and that a fold on a record whose `## Goal` is a fenced board body rewrites that body.
  The directive treats a LOOM `[creel]` line with the replace notice's opening as the parent's instruction.
  `PATTERN-run-records` and the pawl recipe and project-record texts name `replace` beside `add`.

## Design decisions

- No fixer folds, which departs from issue #114's first expected point.
  An entry added in a Bouncer's settling window after the last fixer round would be removed without being folded, so the fold is the parent's explicit write.
- `replace` takes no `--amends-plan`.
  A call that amends an approved plan is recorded with `add --amends-plan` and folded by a later `replace` after the last Webster row.
- The Webster-span refusal exists because a Merriam strand reads no record, and each fresh Master prompt renders only `Added after Discussion` entries.
  The parent directive tells the parent to record each folded call again with `add` before sending a run back to a Webster row.
- The `Lym-Record-Replaced` trailer guards against a mistaken writer, not a forged commit.
  An entry recorded before the newest replace and absent from it is no longer a finding.
  The replace that dropped it stays in git history with the caller's `--by`.

## To check by hand

- `add`'s decision-lock refusal: confirm a refusal-table row in `internal/loomcli` reaches it.
  The last review left this as an open finding (`add-lock-refusal-untested`), and a later commit added a row.
- `replace`'s refusals in `internal/loomcli/decisionreplace_test.go`: confirm each reaches its refusal, the failed-write and failed-restore ones included.
  The last review left this as an open finding (`replace-refusals-untested`), and later commits added tests.
- A fold after the last Webster row followed by `lym loom goto --to` a Webster row leaves the folded call out of every Master spawned there.
  The directive covers a goto the parent makes, but not one the operator makes after a Stuck halt.
- `lym board promote` is not refused while a motif run's weft is live, so a raddle run started in that window adopts the weft and keeps its committed record.
  The next board copy then writes that record over the stored one.
- A replace during a live agent session, such as Plan-Write or Raddle-Write, reaches the agent only through the creel notice.
  A replace made while a Bouncer round is open does not count as a decision made since that round.
- The run report counts post-Discussion decisions from the added entries a record still holds, so a folded call drops out of its count.
