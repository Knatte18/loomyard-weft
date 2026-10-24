---
round: 1
focus:
  - "internal/burlerengine/cluster.go: does the lexical directory resolution (`cd` segments, then `-C`, anchoring on `/`, `~`, `$`) match the decision record, and can any unresolvable or relative path be wrongly matched to a recorded repository (prefix siblings, `..` segments, trailing slashes)?"
  - "internal/burlerengine/cluster.go: are the three bounds enforced (`git push`, `--git-dir`/`--work-tree`, bare `cd`), and is `git init` / `git init ..` never recorded as a fork repository?"
  - "internal/burlerengine/cluster.go: is repository recording done in transcript order, with `git worktree add` recorded only from inside an already recorded repository, and does each command start from the unknown starting directory?"
  - "internal/burlerengine/cluster_test.go: does every acceptance criterion have a row, do all existing matrix rows keep their verdicts, and are any new rows duplicates of existing coverage?"
  - "internal/burlerengine/doc.go and the error message: does the fork-discipline paragraph describe the scoped rule, and is the real-violation message and way forward unchanged?"
---

Seed focus for the fork git-mutation guard change.
Entries name places and questions only.
