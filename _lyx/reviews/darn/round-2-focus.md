---
round: 2
focus:
  - "In `commandRunsMutatingGit` in `internal/burlerengine/cluster.go`, settle whether the `-h`/`--help` escape can be triggered by a word that is an option value (`git commit -m -h`, `git tag -m --help v1`) or follows a bare `--` (`git checkout -- -h`) and so lets a mutating call pass."
  - "In the segment tokenizer in `internal/burlerengine/cluster.go`, settle whether quoted values of `-c` and `-C` (`git -c \"user.name=A B\" commit`, `git -C \"/path with space\" commit`) are kept as one word so the subcommand is still found."
  - "In the segment splitting in `internal/burlerengine/cluster.go`, settle whether a backslash-newline continuation between `git` and its subcommand still yields one segment, so a multi-line mutating call is classified as mutating."
  - "In `gitValuedGlobalOptions` and `TestCommandRunsMutatingGit` (`internal/burlerengine/cluster_test.go`), settle whether every valued git global option, including `--attr-source`, `--git-dir`, `--work-tree`, `--namespace` and `--config-env`, in both separate-word and `=` forms, has its value skipped and a covering row."
---

Focus targets the sites where the segment classifier lets a mutating git call pass or lacks coverage for a valued option form.
