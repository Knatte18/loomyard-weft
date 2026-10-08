# webster record-batch rejects abbreviated head_sha that is a prefix of HEAD

`record-batch-short-sha` · Note · bug, webster

Imported from [issue #456](https://github.com/Knatte18/loomyard/issues/456).

`lyx webster record-batch` refused a batch report because the report's `head_sha` was an abbreviated SHA (9 characters) while the worktree's HEAD is the full 40-character SHA.

Refusal text (paths trimmed):

```
webster: batch report _lyx/webster/reports/01-carry-over-write.yaml: head_sha "ae24b9bb0" does not match the worktree's actual HEAD "ae24b9bb0eca5384fc9fd38764b9cb446aced655"; only merge commits, such as a parent merge-in, may sit between a fork's reported head and HEAD; way forward: 1) run `git reset --keep ae24b9bb0` to move HEAD back to the report's head_sha; 2) re-run this verb
```

Why this is abnormal: the abbreviated SHA is a prefix of HEAD, so the report names the right commit.
The comparison looks like an exact string match, so it reports a mismatch and then suggests `git reset --keep` to the commit HEAD already is, which is a no-op and does not resolve the refusal.
A fork writing `git rev-parse --short HEAD` into the report is a natural thing to do.

What to do: resolve the report's `head_sha` with `git rev-parse` (or compare by prefix) before comparing, and/or require and document the full SHA in the report schema and the fork prompt.
The "way forward" text should not suggest a reset when the reported SHA already resolves to HEAD.

Provenance: task `review-loop-convergence`; notes: webster-refusal-record-batch-2.md
