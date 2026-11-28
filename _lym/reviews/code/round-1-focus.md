---
round: 1
focus:
  - "internal/gitexec/transport.go namesMissingLocalPath: which strings besides a local repository path can git quote in the matched lines (an unconfigured or mistyped remote name, an scp-style or ssh URL, a URL rewritten by insteadOf, a `~` or percent-encoded file URL, a path holding an apostrophe), and how does each one classify once it is stat-ed against this machine?"
  - "internal/gitexec/transport.go namesMissingLocalPath: is the path resolved against the directory git itself resolved it in, for relative remote URLs, `-C` global options, linked worktrees, a run from a subdirectory, an empty GitError.Dir and clone? Check each against real git behaviour, not only the hand-built stderr rows."
  - "internal/gitexec/transport.go: do the matched stderr lines hold under the environment gitexec runs git in (locale), and across push, fetch, pull, ls-remote and clone, as the pattern's doc comment states? Which of these forms does a test pin with real git output?"
  - "Every caller of gitexec.IsTransportFailure, inside and outside internal/fabricengine (including internal/shedtransient and the add, pull, executor and push-classification paths): what changes for each when a missing local path now counts, is any of those changes wrong or untested, and does the issue's 'both places' behaviour hold end to end?"
  - "Docs, comments and specs that describe what counts as a transport failure or an unreachable origin (including the missing or unmounted directory claim in doc.go and transport.go): does each still match the code, including a mount point or parent directory that still exists after the repository is gone?"
  - "_lym/landing/summary.md: check its claims against the diff, including whether `Merge danger` has the right door call and a blast radius that names everything the changed classification can affect."
  - "Existing tests whose fixtures changed from a missing path to an existing non-repository directory (cleanupcli, add_remoteleftover, originreconcile, pushanchored, pushclasses): does each still check what its name and comment say, and is any other test or fixture still relying on the old classification of a missing path?"
---
Seed focus for round 1.
The answer key is a decision record alone, holding the board entry with no `Added after Discussion` entry,
and it leaves the choice between treating a missing local remote as unreachable and documenting that only network failures count.
The entries above are questions to settle, not findings.
