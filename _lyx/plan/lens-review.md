# Lens review

- test-economy, LOW, card 2 `git diff *.go` row repeats `git log *`: rejected; the acceptance criterion names this command, and the row pins it for the `diff` form beside its `git diff -- *.go` counterpart.
- test-economy, LOW, card 2 dollar and parenthesis rows share one branch each: rejected; each is an acceptance-criterion command, and the follower classes differ.
- test-economy, LOW, card 2 `grep 'a$' f` near `grep '$(x)' y.txt`: rejected; the criterion names it and it pins the regex-anchor case directly.
- test-economy, LOW, card 3 fail row repeats `README fails` and the `root_doc_admits_records_repo_name_only` subtest: rejected; the criterion names it, and it pins the fail beside the name at the scan level.
- test-economy, LOW, card 1 `| burler | 2 |` asserted for three slugs: rejected; `live` guards the in-flight bound and `done` the finished bound without a clamp change, at no extra fixture cost.
- criteria-coverage, LOW, card 1 `early` on the `older` commit unspecified: fixed; the card says `older` carries only the tag.
- criteria-coverage, LOW, card 2 `doc.go` criterion has no test row: rejected; it is prose, checked by review.
- criteria-coverage, LOW, card 3 pass row passes with or without the strip: rejected; the criterion asks for the scan's outcome, and the removal itself is the card's edit.
- forced-edits, LOW, card 2 `readOnlySegment` doc comment goes stale: fixed; the Intent now updates it.
- row-reachability: no findings.
