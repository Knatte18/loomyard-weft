```yaml
task: webster-reset-reopens-dead-batch
notes_read:
  - loom-halt.md
issues_filed:
  - https://github.com/Knatte18/loomyard/issues/869
```

`loom-halt.md` showed a shuttle start at `Darn-Write` failing on a transient tmux `fork/exec` EAGAIN, read as a dead provider, with the strand left behind.
Filed as one issue.
