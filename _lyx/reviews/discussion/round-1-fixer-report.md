# Round 1 fixer report

All nine findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` unchanged.

## Changed

- F1 (D2): Bound now says a directive counts only directly above its test function, a duplicate or orphan directive is an error like an empty reason, a kept test stays in the covering pool and removable computation, and a directive on a non-candidate is listed under "Kept".
- F2 (D3): Added a Bound: splice only when the tag set equals the header's, remove a section only when the package lists no tests, keep the old section and exit non-zero on a list or build failure, write a missing `-out` file whole.
- F3 (D1): Bound now says the scan does not detect a hook, alias or helper that makes git run module code, and D4's hand check covers it, keeping such a test under D2.
- F4 (acceptance): third bullet reworded to "no test in an untagged file is listed with the spawns or unclassifiable reason".
- F5 (acceptance): added scenarios for the D1 exclusion triggers and the tagged unresolved call, the misplaced or duplicate directive, and the D3 removal, header, build-failure, tag-mismatch and missing-file cases.
- F6 (D1, D7, Open risks): removed the 483/212 tally, "nine in loomcli" and the package names from the D1 growth risk; the support log already holds the correct per-package figures.
- F7 (D6): replaced the permissive wiring sentence with an instruction keyed on the last `integration`, `tmux` or `llm` file, and kept `HermeticGitEnv` for packages that spawn git.
- F8 (D4): stop condition now reads "no candidate outside the Kept list remains".
- F9 (acceptance): docs bullet now includes the `cmd/testtiming` package doc and D3's splice.

## Deferred

None.
