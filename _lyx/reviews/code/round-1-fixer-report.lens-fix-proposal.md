```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals behind F1 (doc.go amends-plan wording) and F2 (overview.md notice wording).

No findings.

- F1: `beginAmendment` loops over `titles` in order and calls `appendAmendment` for each. That matches "title of each entry … a `--from` batch appending every title in input order". The sentence "one commit lands the entries and the amendment record" matches the batch's single commit. The later sentences of the section (restore on failure, the commit pathspec) read the same under the new wording.
- F2: "the heading of each entry" is correct for a single call (one entry) and for a batch, so it contradicts nothing. The same bullet's `--from` sentence says the notice names the new headings, and the two now agree.
- One adjacent point, out of scope: the `--amends-plan` bullet in `docs/overview.md` still says "appends the entry's title" in the singular. The review did not name it and the fix did not touch it. I rate it LOW, as a stale twin of F1: the F1 fix changed the doc.go sentence, and this overview sentence is its counterpart.

Tool calls:
1. `grep -n -A25 "func.*beginAmendment" internal/loomcli/*.go`: I read the code to check the batch-append claim. My context lacked `decision.go`.
