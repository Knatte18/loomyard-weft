# Keep self-report evidence durable and flag what a board import loses

A board entry imported from a GitHub issue could point at files in a run directory under `.lym/`, which is deleted with its worktree, and nothing said so.
An issue with an empty body became an entry that looked complete but held only its title.
This change closes the gap at both ends: an issue can carry its evidence inline when it is filed, and an import flags what it could not bring along.

## Filing: `lym selfreport create --attach`

`--attach <file>` (repeatable) appends each file's contents to the issue body as a `## Attached` section in a code fence.
The fence is longer than any backtick run in the file, so a file holding its own fences cannot close it early.
The section names the file by its base name only, which keeps a local directory such as a home path out of the public issue.
A body the attachments push past GitHub's 65536-character limit is refused before any request, with an excerpt as the way forward.

The friction reflection stencil now tells the reflection agent to carry run-directory evidence inline, through `--attach` or by quoting the lines, and never to file an issue with an empty body.
An attachment obeys the stencil's existing content rule for the public repository like the rest of the body.

## Importing: `lym board intake import`

`boardengine` adds an import note after the link-back line in two cases:

- a blank issue body gets a note naming the issue title, so a fold, whose section carries no title, keeps it too;
- an issue whose body or comments name paths under a worktree's `.lym/` directory gets a note listing them and saying only what the issue quotes of them reached the board.

`ImportResult` carries `EmptyBody` and `LocalPaths`, and `intake import` prints `"empty_body":true` and `"local_paths":[…]` only when the gap is present, so existing output is unchanged.
`_lym/` paths are tracked and durable and are not flagged.

## Design decisions

- The import flags local paths instead of copying the files.
  A `.lym/` path in an issue is relative to some worktree on whichever machine filed it, and the importer cannot tell which; by import time the worktree is usually gone.
  The durable copy happens at filing time through `--attach`, and the import carries the issue body, attachments included, onto the board verbatim.
- Detection reuses `lymdirs.DotLymDirName` for the directory name rather than a literal.

## Tests

- `TestImportIssueNewEntry` gains rows for a blank body and for local paths in the body and a comment, including trailing punctuation, duplicates and a `_lym/` path that must not match.
- `TestIntakeImport_FlagsEvidenceGaps` pins the two output keys and their absence.
- `TestRunCreate_Success` gains rows for attachments after a body, an attachment as the whole body, a widened fence and base-name naming;
  `TestCommand_RefusedBeforeTransport` gains a missing attachment and an oversized one.
