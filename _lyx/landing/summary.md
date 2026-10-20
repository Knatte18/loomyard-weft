# A session edits nothing until every lens fork has returned

A writer that spawned its lens forks could start fixing on the first fork's report, before the rest had reported.
Editing early saves little, since the forks run in parallel and the wait is about the slowest fork's time.
It costs the whole view: duplicates across lenses cannot be merged once the first is fixed, a later finding cites text that has since changed, and of two findings proposing different changes at one spot the first applied tends to stay.

Each stencil whose session spawns review forks and acts on their findings now states the rule outright: edit nothing while any fork is outstanding, start judging only once the last fork has returned, judge all findings together, then edit.

- `twill-directive.md`, step 3: the writer's wait step carries the rule.
- `burler-fix-self-check.md`, step 3: the fixer's self-check carries the same rule.
- `burler-cluster-fan.md`: the cluster handler writes nothing to the review file until the last fork has returned, then consolidates.
  Its own holistic review while the forks run stays allowed.
- `internal/twilldirective/doc.go` and `internal/burlerengine/doc.go` describe the rule.
  The cluster phase list in `burlerengine/doc.go` is also re-broken to one phase per line.

No issue is closed by this change.

## Design decisions

- The rule sits in the wait step, where a session decides what to do with an early report.
  The judge parts stay host-neutral and unchanged, since they already judge every finding.
- "Edit" covers every write to the files the session reviews or fixes, and for the cluster handler the review file.
  Reading code while forks run stays allowed, since it changes nothing a finding cites.
- The relaunch and resume rules stay as they are: a lost fork never returns, so waiting for it would never end, and those rules already name it as "no report".
- Out of scope: Webster's implementer forks, which run one per batch and are not review forks, and `docs/research/`, which records past runs.

## To check by hand

- The rule is prompt text only, and no gate checks that a session edited nothing while a fork was outstanding.
  Watch a forking writer, fixer and cluster handler on a real run to see that they hold their edits until the last fork returns.
- The change was not run end to end: it is stencil text and doc comments.
