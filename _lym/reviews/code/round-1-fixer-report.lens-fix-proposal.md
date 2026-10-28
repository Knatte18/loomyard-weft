```yaml
lens: fix-proposal
outcome: reported
```

1. LOW: `contracts/stencils/burler/burler-step-1-explore.md`, the moved sentence "Read the files of your focus, or of the whole target when no focus is set, in one message." (F2)
   - It clashes with the fork round's own rule in `burler-fork-reading.md`: "Read definitions and call sites, not whole files". A fork-round handler can read "the files" as licence to read whole files.
   - Narrowing the wording would remove the clash.

Checked, no finding: F1, F3, F4. F5: item 2 has an awkward second semicolon, stylistic only. The `template_test.go` comment revert is correct.

Tool calls:
1. `grep` for `fork_reading` in the burlerengine Go files and the old "then check them" wording.
2. `cat contracts/stencils/burler/burler-fork-reading.md`.
