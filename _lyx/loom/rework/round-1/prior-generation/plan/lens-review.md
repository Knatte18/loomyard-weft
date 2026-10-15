# Lens review

- criteria-coverage, MEDIUM, most D3 paths and fork0 prompt lines unpinned: fixed; card 2's `twill-writer-fork.md` claims row now holds one claim per D3 path and per fork0 prompt line, each phrase kept verbatim in the stencil.
- row-reachability, MEDIUM, same D3 path gap: merged with the criteria-coverage finding above, fixed there.
- row-reachability, LOW, claim "never call the Agent tool" mismatches the capitalised fork0 line: fixed, the claim is spelled "Never call the Agent tool".
- row-reachability, LOW, no row shows `Directive` reads only one part: fixed, `TestDirective_MissingStencilNamesIt` gains two rows that render cleanly with the other part's stencil removed.
- criteria-coverage, LOW, no missing-value rows for `format_step` and `self_check_step` in the parts: fixed, card 2 adds a writer-fork `FormatStep` row and a relaunch `SelfCheckStep` row; the older `rubric_path` and `disposition_place` gaps predate this plan and are out of scope.
- criteria-coverage, LOW, two-advisor loop asserts only one judge phrase: rejected; `DiscussionTable` renders one directive value whatever the advisors (`internal/loomengine/discussiontable.go:96-102`), so one host-value phrase proves the whole block is present.
- criteria-coverage, LOW, `TestPlanTable` lacks the D5 boilerplate: fixed, card 1 wants the Plan boilerplate line with the overview path.
- test-economy, LOW, card 1 host tests repeat stencil-only judge wording: fixed, host tests now pin host values and the removed sentence only.
- test-economy, LOW, card 2 `TestPlanTable` repeats stencil-only writer-fork wording: fixed, those wants dropped; "Spawn one fork, fork0" stays for the ordering check only.
- test-economy, LOW, lensKeys advisor assertion repeats the existing heading check: fixed, dropped.
- test-economy, LOW, card 2 called a card 1 want "kept", and it repeats the relaunch claims: fixed, the Discussion test relies on its existing `Step 6` relaunch want instead.
- forced-edits: no findings.
