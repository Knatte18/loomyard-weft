```yaml
lens: fix-proposal
outcome: reported
```

Findings (lens fix-proposal):

1. **LOW** — `internal/boardengine/doc.go:472`, the F3 rewording.
   - The line is 126 characters of text after `// `. The prose and Go-comment rules cap a line under 120, and the recent gate-fix commits cut lines at exactly this limit.
   - The fix applied the reviewer's wording as one clause. Split it at the semicolon into two lines, for example `It names `lym loom decision add <slug>`;` and `only that verb or `lym loom decision replace` changes the project's scope after that.`
   - The `runlock.go:67` line is 113 characters and passes.

2. **No finding** — F1.
   - I built the failing case: add's `lockRecord` fails after the `--by` and caller checks, so the default fake reaches it.
   - `inputRefusal: true` correctly asserts `appends == 0`, and the shared assertions cover commits, posts and the unchanged record.
   - The row's error text contains "decision lock", and `decisionWayLock` supplies the other two fragments.

3. **No finding** — F2.
   - Row 1: `writeFirstErr` fails write 1, and the restore (write 2) succeeds. That yields the `replaceWayRetry` message, and the record stays at `replaceBaseRecord`.
   - Row 2: `writeAllErr` fails both writes. That yields the restore-failed message with `decisionWayRestoreRecords`, and the file is never written, so the unchanged-record assertion holds.
   - `cmp.Or` over `error` values is safe here, since the dynamic types are pointers from `errors.New`.
   - Counting `recordWrites == 1` as the first write holds, because the replace verb's first `writeRecord` call is at `decision.go:1334`.
   - The `restoreErr` path, which fails only from the second write on, is unaffected.

Tool calls:
- 1 Bash call: measured the text length of `doc.go:472` and `runlock.go:67`, because the edited line lengths were not in my context.
