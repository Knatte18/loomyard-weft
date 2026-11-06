```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The three edits agree with each other and with their neighbours:
- **`diffmode.go:40` and `doc.go:20`:** both now say "an argument that ends a quoted shell string". That matches the `diffmode.go:30` comment on `skipFileArgument`, which was left unchanged and already says the closing quote is never taken.
- **New test row:** it removes the non-final argument and pins `sh -c 'docker run img'`. It sits beside the two ending-quote rows that pin the case the docs now describe.
- **Repo-wide grep:** no other doc, comment or test outside these three files repeats the old "inside a quoted shell string" wording.

Tool calls:
1. Ran a repo-wide grep for "quoted shell" and `sh -c '`, to find twin wording in other files. My context only held the three changed files.
