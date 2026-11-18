# Flag lost issue evidence on import; add --attach to selfreport create

Closes #148

A board entry imported from an issue could name evidence that no one could read.
The issue pointed at files under a run's `.lym/` directory, which is deleted with its worktree.
An issue with an empty body imported silently as an entry holding only a title.
This change flags both at import and lets `lym selfreport create` put the evidence in the issue in the first place.

## What changes

- `lym board intake import` adds `"empty_body":true` to its output when the issue body is blank.
  The entry body gets an import note naming the title as the only content the issue carried.
- It adds `"local_paths":[…]` when the issue body or comments name paths under a `.lym` directory.
  The entry body gets an import note listing them, saying the files are deleted with their worktree
  and that only what the issue quotes of them reached the board.
  Both flags apply to a fold (`into`) as well as a new note.
  The `board` skill, `boardcli` help and `boardengine` docs describe the flags.
- `lym selfreport create` takes a repeatable `--attach <file>`.
  Each file is appended to the body as `## Attached <base name>` followed by its contents in a code fence.
  Only the base name is used, so a local directory path does not reach the public repository.
  The fence and the name's code span are lengthened past any backtick run they hold.
  A non-UTF-8 file, or a body pushed past GitHub's 65536-character limit, is refused with the way forward named.
- The friction reflection stencil tells the agent to attach or quote run-directory evidence
  and never to file an issue with an empty body.
  `docs/overview.md` records the flag.

## Design decisions

- Import flags missing evidence and does not try to copy it.
  The issue text only names the paths, and by import time the files are usually gone, so there is nothing to copy.
  The fix for the source is at issue creation, through `--attach`.
- Detection is a regexp over the issue body and comments for a path that starts with or contains `.lym/` or `.lym\`.
  It stops at markdown delimiters and strips trailing punctuation and emphasis markers.
  The paths come back distinct, in first-mention order.
- A body counts as empty when it is only white space.

## To check by hand

- `import-copies-no-named-evidence`: the import still copies none of the files an issue names.
  It only flags them, in `internal/boardengine/intake.go`.
  Confirm that flagging meets the issue's first "What to do" item, or whether copying is still wanted.
- `attach-breaks-provenance-last`: `--attach` appends after the body, so the provenance line the stencil asks for
  is no longer the last text of the issue.
  The stencil now says the attached sections follow it; check that wording in
  `contracts/stencils/friction/friction-template-reflection.md`.
- `fold-gap-handling-untested`: the flags on the fold path rest on the shared code in `importIssue`
  in `internal/boardengine/intake.go`; check that the fold tests there cover them.
