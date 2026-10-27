```yaml
lens: fix-affected-set
outcome: reported
```

Checked every finding whose premise is a set. None left unchecked.

- **F1** (callers of `Migrate`): all of them are covered. The full-file reconcile at `configsync.go:507` is the only one card 11's Intent does not name. Gate never reaches it because the module is `HubWide`, and `MigrateConfig` is now unchanged anyway.
- **F5** (readers of `ReadCheck`): both are covered. Card 14's `ensureCheck` is the other reader, and it runs only where the check directory gets written anyway.
- **F7** (record reads after a tick in cards 8 and 9): the kept-tidy refresh is the only one, and it is fixed.
- **F10** (docs that list the gate keys): one member is missing, below.

Finding:

- LOW. Changed text: card 11's `**Extends:**`, which now lists `pattern/PATTERN-test-speed.md`, and its Intent bullets on the docs that list the gate keys.
  The set of docs that spell out every gate key also contains `docs/dotnet-hub.md:88-98`. That file has an example `gates:` block that lists every key lym reads, through `finalize`, and the fix gives it no `adopt` line. The `internal/gatecli/doc.go` example block does gain one.
  Evidence: grepping for `parent-align` across `.md`, `.go` and `.yaml` files turns up `docs/dotnet-hub.md:97-98` (`parent-align: …`, `finalize: {tiers: [0], scope: all}`), and no card names that file. `contracts/specs/loom-plan-spec.md:318` names keys by site in prose and is not a full key list, so it needs no change.

Tool calls:
1. grep for `.Migrate(` callers in `internal`, to rerun F1's caller search, which my context had only as configsync excerpts.
2. grep for files naming `parent-align`, to rerun F10's search for docs that list the gate keys.
3. grep for `ReadCheck` in the plan cards and adopt-directory doc comments, to rerun F5's and F10's reader and doc sets.
4. grep for `parent-align` lines in the candidate files, to see which ones list the keys in full.
5. read `docs/dotnet-hub.md:88-106`, to confirm the example block lists every key.
6. grep for state reads in cards 8 and 9, to rerun F7's search for post-tick reads.
