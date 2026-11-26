# Keep each landed run's findings, asks and fixer reports in its archive tag, with an index row and lym report landed

`keep-landed-run-records` · Task · feature, landing, fabric, report, loom, burler, shed, cwd, git, testkit, pattern · draft

Decision record: [Sketch](decision-keep-landed-run-records.md)

Depends on: [`pr-modes-no-agent`](design-pr-modes-no-agent.md), [`describe-review-and-sync`](design-describe-review-and-sync.md), [`push-retry-not-halt`](design-push-retry-not-halt.md), [`beam-member-parent`](design-beam-member-parent.md), [`scope-and-earlier-checks`](design-scope-and-earlier-checks.md)

Filed from the Sketch `keep-landed-records`.

Takes: the Sketch's decision record whole: the archive tags teardown pushes, the loom status commit and run-records set, the parent-review ask record's locked updates in internal/burlerengine, the landing fixer and fix review scratch reports, and Finalize's merge-in, merge and push steps
Delivers: one kept directory under each run's _lym/ that the status commit stages and the archive tag carries, holding PR findings copies, ask record and report copies seeded back after an adopt, landing fixer and fix review report copies, the landing record and the index row; and a read-only lym report landed verb that lists and extracts every archived run's kept records
Exports: the kept directory's name in internal/lymdirs; the landing record and index row types with their writers and one reader in internal/shedrun; the first-parent landing searches in internal/fabricengine; pullrequest.API.Merge returning the merge commit SHA; the cobra verb lym report landed

The part changes these packages:

- internal/lymdirs (about 10 changed lines)
- internal/loomengine (about 20 changed lines)
- internal/loomcli (about 250 changed lines)
- internal/fabricengine (about 200 changed lines)
- internal/burlerengine (about 250 changed lines)
- internal/landingshed (about 450 changed lines)
- internal/pullrequest (about 30 changed lines)
- internal/pairteardown (about 5 changed lines)
- internal/testkit/prkit (about 5 changed lines)
- internal/shedrun (about 140 changed lines)
- internal/gitrepo (about 120 changed lines)
- internal/reportengine (about 430 changed lines)
- internal/reportcli (about 170 changed lines)
- PATTERN.md (about 4 changed lines)
- pattern (about 20 changed lines)
