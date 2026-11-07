# worktree list read in-process through internal/dotgit

`dotgit-worktree-reader` · Task · enhancement, fabric, testkit, git · done

Decision record: [Sketch](decision-dotgit-worktree-reader.md)

Filed from the Sketch `fabric-test-speed`.

Takes: dotgit's FindRoot and Read, fabricengine.List's porcelain parse, and websterengine's otherWorktrees
Delivers: a stdlib-only dotgit worktree reader matching git worktree list --porcelain, with a named refusal of a non-files ref storage; List and otherWorktrees read through it and fall back to porcelain on that refusal; the PATTERN-leaf-packages background lists the worktree entries and refs dotgit parses
Exports: the dotgit worktree reader, its entry type and its ref-storage error

The part changes these packages:

- internal/dotgit (about 650 changed lines)
- internal/fabricengine (about 100 changed lines)
- internal/websterengine (about 30 changed lines)
