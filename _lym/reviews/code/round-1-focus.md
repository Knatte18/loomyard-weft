---
round: 1
focus:
  - "internal/dotgit/worktrees.go: does ListWorktrees match git worktree list --porcelain on ordering (main first, linked sorted by path, case-folded only when core.ignorecase is true), the main path resolved through symlinks, and a relative gitdir resolved against its worktrees/<id>/ directory?"
  - "internal/dotgit/worktrees.go: are the omission and prunable rules right — entry dropped when gitdir is missing, unreadable or empty; prunable only when not locked and the named .git is missing; no HEAD file listed detached with the all-zero SHA; a ref missing from both the loose ref and packed-refs gives the all-zero SHA?"
  - "internal/dotgit/worktrees.go: is HEAD resolution correct for a symbolic ref, a detached SHA, and a branch only in packed-refs, and are the failure paths (unreadable files, malformed packed-refs, bare main repository still an error) handled without a panic or a silent wrong answer?"
  - "internal/dotgit/worktrees.go and internal/gitkit/query.go: a directory that does not exist is an error with no walk-up, while a subdirectory of a worktree finds its repository by walking up; does the gitkit/query.go change fall inside the answer key's scope, and does it keep the fallbacks in fabricengine junction.go and scratchlink.go firing?"
  - "ErrRefStorage: is extensions.refStorage read correctly (including a missing config key and a value other than files), and do fabricengine/worktreelist.go and websterengine/gitwrap.go each fall back to git worktree list --porcelain on that error alone, with no other error swallowed into the fallback and no git process started on a files-backend repository?"
  - "Scope and PATTERN: does the diff stay inside internal/dotgit, internal/fabricengine and internal/websterengine, keep dotgit standard-library only and spawn-free, and land the PATTERN-leaf-packages change (pattern/PATTERN-leaf-packages.md) with the declared text, including the background's list of what dotgit parses?"
  - "Tests against the acceptance criteria: is each D1 criterion covered once (parity incl. locked with and without reason, prunable, symlinked path, removed gitdir, removed HEAD, relative gitdir with a version skip, packed-refs only, reftable fallback for both callers, no git process, subdirectory and missing directory), and do any new tests duplicate worktrees_test.go and worktrees_git_test.go coverage or assert static prose?"
  - "Commit messages and doc comments: do the commit messages and doc.go changes in dotgit, fabricengine and websterengine match the diff, and do new doc comments follow the repository's comment conventions?"
---

Round 1 focus set from the decision record's D1 and its acceptance criteria, and from the diff stat of the review range.
