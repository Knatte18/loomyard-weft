# Lens review

Every fork returned a report.

- criteria-coverage, MEDIUM, card 4: the doc-comment criterion had no check — fixed: the card prescribes the wording "ref-update race" beside the quoted refusal line and adds a `Verify:` grepping `pushretry.go` for both.
- criteria-coverage, MEDIUM, card 6: no row refused more than one file in one call — fixed: a row names a non-`.go` path and an unparsable file together and asserts `refused` lists both, with nothing written.
- criteria-coverage, LOW, card 6: the Intent said `wireLightweight` fills `c.cwd` — fixed: the pre-run's `lyxcwd.CwdFrom` read fills it, as `internal/loomcli/cli.go`'s field comment says.
- criteria-coverage, row-reachability and test-economy, LOW, card 7 (merged): the gate-refused row's seam assertion was vacuous and repeated `TestRemove_RefusesForeignWorktreeWithoutDeletingIt` — fixed: the row is dropped, and that existing test gains a counting seam and asserts it never ran.
- row-reachability and forced-edits, MEDIUM, card 1 (merged): the launch-carried turn's death in `Dies` landed at `AddStrand`, so the pane-dies row ended in the startup probe — fixed: the death is armed for the second `Status` call, and the row also asserts `Started` true in the run record.
- row-reachability and forced-edits, MEDIUM/LOW, card 1 (merged): the hook-state rows and the `hookStateSkillEngine` doc described a typed first load — fixed: `hookStateSkillEngine` joins the card, its doc and both rows are reworded to the hook fold's idle reading after the load turn.
- row-reachability, BLOCKING, card 6: a read-only directory does not stop `os.WriteFile` onto an existing file — fixed: the second file itself is made 0o444, and the row skips only as root.
- row-reachability, MEDIUM, card 7: the Windows row's `os.Create` made the pair dirty, so `Remove` refused before git — fixed: it holds open a file committed on the pair's branch with `os.Open`.
- row-reachability, MEDIUM, card 3: the resize value's timing could be taken by an earlier wait or hang the test — fixed: the resize step's fake `After` sends onto a 1-buffered `Resized` and the row runs under a ten-second context timeout.
- row-reachability, LOW, card 1: the color test's new rows needed table fields and the `EndsAtColor` assertion contradicted the no-color rows — fixed: `skills` and `reports` fields are named, `EndsAtColor` applies only to rows that type a color, and `playLoad` defers a `LateEnds` turn's end.
- row-reachability, LOW, card 6: the symlink row had no Windows skip — fixed: it skips when the platform refuses to create the link.
- test-economy, LOW, card 3: the shorter-narrower row could not fail — fixed: dropped; `rendered`'s byte form on every row carries the criterion.
- forced-edits, MEDIUM, card 7: `fabricengine/doc.go`'s `destroy.go` paragraph and the chokepoint background file say every executor runs the four checks — fixed: both name `removeWorktreeLeftover`'s exception, and the background file joins the card.
- forced-edits, LOW, card 6: `verbUsesLightweightWiring`'s doc comment enumerates the lightweight set — fixed: the Intent adds `fmt` with its reason.
- forced-edits, LOW, card 1: `Claude.Prepare`'s doc and inline comments still said an empty input box — fixed: the Intent rewords both.
