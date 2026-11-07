```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour**

1. `trustedSender` matches on the repository name, case-insensitively, and an empty or shortname entry never matches: pinned. The rows `TaskOrchOfAListedHub`, `AListEntryThatDiffersOnlyInCase`, `AListThatNamesTheShortname` and `AHubWithNoRepositoryName` are in `crosshub_test.go` (~660–690).
2. `Deliver` stores mail untrusted with an empty repository when `RepositoryName` fails: pinned by the `third` message in `crosshub_test.go:653-657`.
3. `HubStamp` and `MessageLine` lead with the repository, and a message with no repository gets the bare stamp: pinned at `crosshub_test.go:643-655`.
4. `ListsRepository` (empty never listed, `EqualFold`): pinned indirectly by the `trustedSender` rows above and the `creel hubs` row below.
5. `record.SendingRepository` is journaled on `inbound`, and `applyInbound` reads it back: **none for the replay**. `journal_test.go:261` and `:802` build `eventInbound` records without a repository, and nothing asserts it comes back from a journal read, so dropping the field from `applyInbound` would go unnoticed. `crosshub_test.go:643` only checks the live in-memory message.
6. `applyCarried` and `carryRecords` carry `SendingRepository` across the retention boundary: **none**. `journal_test.go:802-840` pins `SendingHub`, `SendingID` and `Trusted` through carry, but no row sets or asserts a repository.
7. `messageFields` in `cli.go:326` emits `sending_repository` when set and omits it when empty: **none**. The grep found no test for `sending_repository`. This is a visible envelope key documented in `creelcli/doc.go`.
8. `lym creel hubs` shows `repository`, `no_repository` with a reason, and `trusted` by repository: pinned by the rewritten `cli_test.go` row at ~468–487. It covers a trusted case-insensitive match, a hub whose shortname is listed but whose repository is non-GitHub (`names no GitHub repository`), and a hub with no origin (`no readable origin remote`).
9. `hubregistry.RepositoryName` and `fabricengine.HubOriginURL`: pinned only through the `creel hubs` row (the prime path derived from the hub path, plus the error branches). `ParseOwnerRepo` shapes belong to `githubclient`. No `hubregistry` unit test exists and none is needed beyond that row.
10. `hubregistry.LookupRepository` and `configreg.repositoryOfShortname`: **none**. The only test mentioning them is `reinterpret_test.go`, which uses a scripted resolver. The real lookup, including a shortname that resolves to none or several hubs, is never run. This is the production resolver for the reinterpretation.
11. The `creel` module registers `TrustedHubsRepositoryReinterpretation` in `configreg.Modules()`: **none** for the registration. `configreg_test.go:210` is a generic row that still uses the shortname fixture `ly`, and nothing asserts the reinterpretation is wired up. Dropping the `Reinterpretations:` entry would pass every test.
12. `RewriteShortnamesToRepositories`: pinned by `reinterpret_test.go`. Rows cover rewrite, drop with reason, a repository kept, case-insensitive dedupe, no change, an empty list, no key, idempotence on a second run, and a malformed file returning `ErrInvalid`. A non-scalar entry in the list is not covered, which is minor.
13. `hubDirectory.RepositoryName` in `creelcli/hubdirectory.go`: not unit-tested. `hubdirectory_test.go` has no row for it. Every `creelcli` test calls it only through the `hubs` verb (which uses `hubregistry` directly), so the daemon's real directory path for `Deliver` is never run. Any `creelcli` test that delivers cross-hub mail would reach it, and none was found.
14. `HubDirectory.TrustedHubs` reads repository names: `hubdirectory_test.go:90-104` still uses the fixtures `aa`, `bb` and `cc`. This is stale but still correct, because that read does no validation.
15. `creelconfig` `HubValue` row: updated to repository names.

**Findings**

- F-A, MEDIUM, design. Location: `internal/creelengine/journal_test.go:802-840` and `internal/creelengine/fold.go:518,600,706`. Nothing pins `SendingRepository` through journal replay (`applyInbound`) or retention carry (`applyCarried`, `carryRecords`). The field is journaled but the tests only check the live message. Fix: put a repository on the inbound records of the existing carry and replay journal tests and assert it survives, as rows in those tests.
- F-B, MEDIUM, design. Location: `internal/creelcli/cli.go:326-328`. The `sending_repository` envelope field is set when present and omitted when empty, and no test asserts either. Fix: add an assertion to the existing cross-hub read test in `cli_test.go`.
- F-C, MEDIUM, design. Location: `internal/configreg/configreg.go:118-125,164` and `internal/hubregistry/repository.go:659-667`. The production wiring of the reinterpretation (the registration, `repositoryOfShortname`, `LookupRepository`) has no test. The reinterpretation test only covers a scripted resolver. Fix: add a `configreg` row asserting the `creel` module carries `TrustedHubsRepositoryReinterpretation`, and one `hubregistry` row for `LookupRepository`, covering a recorded shortname resolving to its origin repository and an unrecorded one erroring.
- F-D, LOW, design. Location: `internal/creelcli/hubdirectory.go:83-86`. `hubDirectory.RepositoryName` is untested directly. Fix: add a `RepositoryName` row to `hubdirectory_test.go` beside the existing `TrustedHubs` row, using a hub fixture with an origin set.

**Tool calls**

1. One `grep` batch over the test files for `sending_repository`/`SendingRepository`, the `configreg` and `hubdirectory` trusted-hubs rows, `LookupRepository`/`RepositoryName`/`HubOriginURL` callers in tests, and `SendingHub` assertions. My context held the test diff only, not the wider test files.
