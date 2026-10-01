# Rework coverage — PR #321 round 4

- **A. Evidence must cover HEAD, not only the worktree** — cards 56 (accept-audit reconciles HEAD against the last batch head) and 58 (`--fresh` requires HEAD at the run's start commit); spec rows in 64.
- **B. `recover-batch` must not clear findings it cannot check** — card 60 (uncheckable findings recorded on the failed batch, recover-batch refuses toward `run --fresh`, `--fresh` drops such a batch); spec rows in 64.
- **C. A fork must not write webster's state** — card 62 (`fork-state-write` correctness class, terminal fork batches audited before the "already terminal" refusal); spec row in 64.
- **D. Plan edits must not slip into the recorded hashes** — card 59 (validate, record-batch and recovery refuse a changed plan instead of restamping), card 58 (`--fresh` checks plan paths, plan-path way forwards), card 57 (plan baseline store and `lyx webster restore-plan`, the working restore for plan paths); spec rows in 64.
- **E. Pick the evidence base by execution order** — card 55 (last batch head and run start commit picked by git ancestry, rebaseline included).
- **F. `.lyx` writes outside webster's scratch** — card 63 (`filepath.Dir(geom.ScratchDir)` in `runDirs`); spec row in 64.
- **G. Smaller gaps in the same rule**
  - blob looked for anywhere in the head tree — card 61;
  - unresolvable base SHA treated as unverifiable with a way forward — card 55;
  - `refusal-spec.md:61` run-exit policy row corrected to state no re-run at run exit — card 64;
  - spec says the audit sees Write/Edit/NotebookEdit writes only and catches slips, not an adversary — card 64.
