# Fabric: give five undriven failure cases a way forward

A review of the one-repository verbs drove five cases that end in a raw git error, a silent skip or an unnamed command.
This change fixes each, in `internal/fabricengine`, `internal/fabriccli` and `internal/battencli`.

## What changes

- **Killed clone.**
  A `lym fabric clone` killed after its records store exists and before `_bolt` does left a hub no verb finished:
  `lym fabric reconcile` failed with a raw `dotgit: not a git repository`, and a re-clone answered `hub already exists`.
  `reconcile` now refuses such a hub before its first Bolt pull (`RefuseHubWithoutBolt`).
  The refusal, and `CloneHub`'s `hub already exists` error, name the `--reset` clone that replaces the hub.
  For a one-repository hub, the refusal names both the join and the seed; the `CloneHub` error names the join.
- **Unreadable records layout.**
  `lym fabric merge` read the weft SHA from stale tracking refs when the records store's `remote.origin.fetch` was unset, and said nothing.
  `resolveMergeSources` now reads the layout first and returns the error that names the refspecs to restore.
  The same read now guards `fastForwardWeft` (which warns and skips), `FetchPairTips` and `ReturnToPreviousTransition`.
  `catchUpPair`'s fetch in `internal/battencli` returns the layout error as it is, instead of wrapping it as an unreachable origin.
  The error matches the new `ErrRecordsLayout`.
- **Kept local archive tags.**
  `--remote` on `lym fabric remove` and `lym fabric cleanup` already pushes the slug's local `archive/<slug>/*` tags that origin lacks,
  including a tag `DropLocal` kept.
  The push stays as it is: the help of both verbs and `DropLocal`'s doc now say so.
- **Plain archive tags on a one-repository origin.**
  `lym fabric cleanup --remote` lists, as `plain_archive_tags`, each `refs/tags/archive/*` and `refs/tags/archive-owner/*` ref an earlier binary pushed.
  Each entry carries the commands that move the tag under the records namespace and delete the plain ref.
  A failed listing is named in `plain_archive_tags_error` and never changes the exit code.
- **Discard re-run.**
  A killed discard's pending record named "the command that discarded the task branch", which no operator can type.
  `DiscardTaskBranch` takes the command, and the job and the pending record carry it as `rerun`.
  `lym batten cancel <slug> --remove` passes its own command.
  The cleanup path passes none, since the dobby's next tick re-runs the discard, and keeps the described way forward.

## Design decisions

- A local archive tag may leave the machine.
  The smallest change is to document the push the code already makes rather than give `DropLocal`'s tags a prefix the push skips.
- `PlainArchiveTagsOnOrigin` only reports, so the moving commands are printed and never run.
  They push atomically, so a failed move deletes nothing.
- `DiscardTaskBranch`'s signature gains a `rerun` string.
  A job written by an earlier binary has none and gets the old wording.

## To check by hand

- `catchUpPair`'s layout error is returned as is (`internal/battencli/catchup.go`).
  A review judged its earlier wrapping, which reported the error as an unreachable origin, a misreport; confirm the new path reads right.
- The `lym fabric cleanup` help names the records namespace through `fabricengine.RecordsNamespaceRoot` (`internal/fabriccli/fabric.go`).
  A review flagged the namespace's spelling in the help; read the rendered help.
- The layout reads added for the merge, fast-forward, fetch-tips and return-to-transition paths
  (`internal/fabricengine/mergeguards.go`, `parenttips.go`, `codetipsettle.go`) have integration tests for some paths.
  A review found the coverage of these reads thin; check that each path has a test of an unreadable layout.

## Merge danger

**Door:** two-way
**Blast radius:** fabricengine
