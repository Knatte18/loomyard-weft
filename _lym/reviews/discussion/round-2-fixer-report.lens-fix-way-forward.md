```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that each way forward the fix introduced can be carried out from the state it names and leads to progress.

No findings.

How I checked each way forward:
- **D11, failed archive in `lym beam start`:** the refusal names the error, and running `lym beam start` again reaches the same refusal path, which runs the archive again. When the archive succeeds, the refusal goes on as it does today.
- **D11 Bound, `lym s` on the old owner:** `adopt --here` with no record does a Bolt pull, reads the foreign owner and becomes `lym adopt <subject>`. Adopt stops an entry whose branch is ahead, diverged, rewound or rewritten (`internal/adoptcli/adopt.go:747`, `:792-795`). It names `--keep-local` and `--drop-local`, and `--drop-local` archives each local tip. So "meets the tips through its local-commit stop" holds and leads to an actionable step.
- **D2 criterion, a discard that holds the lock:** the caller exits non-zero naming the re-run, as D1 line 275 already says. With the record removed, the re-run starts a fresh adopt.
- **D5, an adopted beam's own `adopt` entry:** it is lifted by `lym tender resume <beam>` on the beam's pair slug, which is a pair in this hub, so tender's refusal of a slug with no pair does not apply.

Tool calls:
- `grep` for keep-local and LocalCommit in `internal/fabricengine`: to find the local-commit stop the D11 Bound now relies on, which my context lacked. It returned nothing.
- `grep` for keep-local files and "local commit" in `internal/fabricengine`: to locate where adopt decides its local-commit stop.
- `grep` in `internal/adoptcli/adopt.go` plus a read of `originreconcilemoves.go:170-200`: to confirm which branch states stop an entry and name `--keep-local` or `--drop-local`.
