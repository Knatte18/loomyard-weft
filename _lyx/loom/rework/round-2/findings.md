# Review of PR #321 (refusal-audit), round 2: fix needed

Items 1–7 from round 1 are fixed, each with a reaching test, and the full integration suite passes.
Three defects remain; fix each with a test that reaches it.

## A. MED — the fabric-reference classifier fails open

`internal/websterengine/audit.go`, `mutatingSegment` (around lines 540–640), treats any program it does not recognise as read-only.
A command that touches the fabric sibling through an unlisted writer is classed policy and only warns, which is the state-rewrite hole round 1's item 2 was about, and a regression against main, where every fabric-reference finding halted.
Examples that pass as policy today: `python3 -c "open('<fabric sibling>/webster/state.json','w').write(...)"`, `dd of=`, `rsync`, `install`, `patch`, `chmod`, `mkdir`, `tar -x`, `unzip`, `awk '{print > f}'`, `bash script.sh` (no `-c`), `eval`, `source`, and `lyx` verbs other than read-only fabric ones.

Fix: invert to an allowlist.
A fabric-referencing command is policy only when every segment is a known read-only program (cat, ls, grep, head, tail, wc, stat, read-only git such as log/show/diff/status/rev-parse, read-only `lyx fabric` verbs) with no write redirect; everything else is correctness.
Keep the existing parser for quotes, substitutions, wrappers, `find -exec` and redirects.

## B. LOW — accept-audit's fabric-sync-failure refusal has no spec row or test

`acceptaudit.go:70` refuses on a fabric sync failure, but `contracts/specs/refusal-spec.md` has no row for it (row 80 lists only the bracket verbs, run and rebaseline) and no test reaches it, which breaks the Refusal Way-Forward Invariant.
Add the row with its way forward and a reaching test.

## C. LOW — pending audit findings fail the run instead of blocking it

`ErrPendingAuditFindings` surfaces from the Webster row as a producer error (`internal/shedadapters/webster.go:96`), so the run goes failed and ly-drive treats it as a repair candidate.
A correctness halt should leave the run blocked with the way forward (`lyx webster accept-audit` after the operator has checked the named paths), which ly-drive hands straight back.
Map it to Stuck/blocked with that reason, and test the resulting state.
