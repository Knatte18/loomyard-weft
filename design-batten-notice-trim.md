# Batten notices: no attach clause, pair-relative paths, wait is not an error

`batten-notice-trim` · Task · enhancement, batten, orch · running · Describe

## Seen (operator, 2026-10-11)

A batten notice to the orch ends with `; cd <absolute pair path> && lyx reed attach` (`internal/battenshed/notice.go`, `renderNotice`), and its `output` and `report` clauses carry absolute paths.
The orch knows how to attach to a pair, so the attach clause is noise, and absolute paths repeat the hub prefix in every notice, which lands in an Opus session that re-reads its context each turn.

Seen (operator, 2026-10-05): shed keeps the reason for every halt in `Status.Error`, an awaiting PR-Gate included, and `renderNotice` labels it `error:` whatever the state, for example `child left running; child state awaiting, error: the pull request awaits review; …`.

## Wanted

- Drop the attach clause from every batten notice.
- Render every path in a notice relative to the task pair, since the slug already names the pair (operator, 2026-10-10), for example `_lyx/shed/<slug>/drive-reports/drive-report-….md`.
- Label the reason by state: `error:` only for failed and blocked, and `waiting:` (or the bare reason) for awaiting and paused (operator, 2026-10-05).
- Phrase the condition by what happened, for example `board-config-per-worktree: awaiting at PR-Gate: the pull request awaits review`, not `child left running; child state awaiting`.
- Update the notice tests and the docs that quote a notice line.
