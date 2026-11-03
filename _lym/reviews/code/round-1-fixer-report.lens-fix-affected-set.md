```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1** (the override bullet): the search for "Follow the rule above over" returns no hit anywhere in the repo outside `.git`, `.lym` and `_lym`. The old wording has no twin, and `shipped.json` is the only place the lym-batching text lives.
- **F2** (step 4 sub-steps): the searches for "Run the gate or validator again" and "Decide every fix." hit only `internal/heddleengine/shipped.json:13`, the line the fix changed. No other file repeats the sequence as unordered bullets.
- **Unchecked:** none.

Tool calls:
1. `grep -rnF` across the worktree for the three old phrases, to confirm no other file carries the old wording. My context held only `shipped.json`.
