lym webster accept-audit refused

Arguments: 

```
webster: pending audit findings cannot be accepted: webster: accept-audit: last batch head: head_sha "fea915351951e0dd58b5903c19a733f681985d7f" does not match the worktree's actual HEAD "57513a96976de817d01d74fadc9650f8e335716c"; only merge commits, such as a parent merge-in, may sit between a fork's reported head and HEAD; way forward: 1) lym webster reset --to last-batch-head; 2) re-run "lym webster accept-audit"
```
