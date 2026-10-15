# tokencount splits Burler roles across the overview-seats rename

`tokencount-role-rename` · Note · enhancement, burler

## Seen

`overview-seats` (PR #624) renamed the Burler seat roles; its decision record names `tools/tokencount` role keys as an open risk.
`tools/tokencount` takes a session role from the strand title, so runs before and after the rename may land in separate rows (e.g. `burler-review-chair` against the new name).

## Idea

tokencount maps the old role names onto the new ones, so a report over runs on both sides of the rename sums each role once.
