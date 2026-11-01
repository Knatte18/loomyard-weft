```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F9, F10, F5, F6 and F1 fixes cover every member of their sets.

**no findings**

- **F9, F10:** a repo-wide search for the old "every card gate runs ... after its tests" and "then the comment line-break lint" wording finds no remaining match.
  - The hits that remain are `webster-body-implementer.md:28`, `websterengine/doc.go:79` and `PATTERN-test-speed.md:11`.
  - Each says "card gate step(s)" or "runs on every card gate". That reads correctly for the gate call, and the review marked the Webster edits as licensed.
  - `PATTERN-gate-slot.md` and `docs/overview.md` already carry the new wording.
- **F5:** `tierResult` has one producer, `runTier` at `tierrun.go:124`, and `tierFailedRun` is the only consumer that builds a `failedRun` from it. `buildFailedRun` and `lintFailedRun` set no causes, which is correct because neither has cause lines. `reportLines` is the only reader of `failedRun.causes`.
- **F6:** `qualify` has one caller, `failurereport.go:235`, and it now passes the whole `runFailure`. Vet and guard "its output" lines stay unqualified, but the finding covered only build-error lines in failures.
- **F1:** the old cut-line text `more lines; the logs` appears nowhere outside the updated test and `doc.go`.

**Unchecked:** F2, F3, F4, F7 and F8 are coverage additions. Their premises are single sites, not sets, so I did no set search for them.

Tool calls:
1. grep for stale "card gate runs" wording across `.go`, `.md` and `.yaml`, because my context lacked the other docs.
2. grep for `qualify`, `unparsed`, `causes` and `tierResult{` in non-test Go, to list every producer and consumer.
3. grep for the old cut-line text and `lint-comments`/`lint-pattern` mentions, to find stale descriptions elsewhere.
