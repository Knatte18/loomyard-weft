# Card 2 — record-batch resolves an abbreviated head_sha

**Create:**
- `plan:internal/websterengine#ErrHeadSHAUnresolved` -> `var ErrHeadSHAUnresolved error`
- `plan:internal/websterengine#commitsNamedBy` -> `func commitsNamedBy(worktree, prefix string) ([]string, error)`
- `plan:internal/websterengine#realGit.CommitsNamedBy` -> `func (realGit) CommitsNamedBy(worktree, prefix string) ([]string, error)`
- `plan:internal/websterengine#resolveReportHead` -> `func resolveReportHead(git Git, worktree, reportPath, headSHA string) (string, error)`

**Edit:**
- `internal/websterengine#Git`
- `internal/websterengine#RecordBatch`
- `internal/websterengine/gitfake_test.go`
- `internal/websterengine/gitwrap_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Intent:**
D1 for record-batch (#456): a report's `head_sha` that abbreviates HEAD is resolved to the full SHA before any comparison, so a fork that wrote `git rev-parse --short HEAD` no longer forces a reset and re-drive.

The git probe:
- `Git` gains the method `CommitsNamedBy(worktree, prefix string) ([]string, error)`: the full SHA of every commit whose object name starts with prefix, sorted, empty when none does.
- `commitsNamedBy` in `gitwrap.go` implements it through the checked `gitexec.Run` (`PATTERN-gitexec-checked-call`): `git rev-parse --disambiguate=<prefix>` lists every object the prefix names, and each listed object is kept only when `git cat-file -t <sha>` answers `commit`.
  Only object names are matched, never ref names, so `HEAD`, a branch or a hex-named ref never resolves.
  It never goes through `gitrepo.Repo.ResolveSHA`, which returns the first match of an ambiguous prefix and also resolves ref names.
  For a prefix naming nothing, git prints nothing and exits 0, so an empty `--disambiguate` answer is no match, not an error.
  The `gitwrap.go` file comment lists the new probe among the read-only helpers it wraps.
- `realGit.CommitsNamedBy` delegates to it.

The resolver, `resolveReportHead`, beside `reconcileReportHead` in `gitwrap.go`:
- a value of exactly 40 hex digits returns unchanged with no git call;
- a hex value of 4 to 39 digits, of either case, is lowercased and passed to `git.CommitsNamedBy`, and exactly one commit returns that commit's full lowercase SHA as `CommitsNamedBy` answers it;
- no commit, or more than one, returns an error wrapping `ErrHeadSHAUnresolved` that names the report path and the value and says whether it named no commit or how many, with the way forward `have the fork rewrite head_sha in <reportPath> as the output of `git rev-parse HEAD`, then re-run this verb`;
- any other value (one holding a non-hex character, or longer than 40) returns unchanged, so it meets the refusal at its consume site;
- a git failure is returned as an error that does not wrap `ErrHeadSHAUnresolved`.
The 4-digit floor is git's minimum abbreviation.
The way forward names only `git rev-parse HEAD`, read-only, for the fork to run (`PATTERN-no-denied-recovery`).

`RecordBatch` resolves once, before the value's first use:
- on the main path, right after `ParseReport` succeeds, `report.HeadSHA` is replaced by the resolved value, so `reconcileReportHead`, `failFromFindings`, `postBatchChecks`, `failCardNotDone`, `failBatch`, the digest's `HeadSHA` and `CardSHAs` all see the full SHA, and the moved-HEAD warning names it;
  a resolution error is returned as it stands, the batch left non-terminal and the report not archived;
- on the correctness-failure path, the parsed report's `head_sha` is resolved too: a resolved value goes into the failed digest, an `ErrHeadSHAUnresolved` error makes the digest head empty, as for a report that does not parse, so the correctness finding wins over the unresolved value, and any other error is returned.

Tests:
- `gitfake_test.go`: `fakeGit.CommitsNamedBy` answers every registered SHA (the `parents` keys) that starts with the prefix, sorted.
  `fakeGit.mint` mints SHAs whose leading digits differ between commits (the hex SHA-1 of the counter, say), so a 9-digit prefix of one commit names that commit alone; a test needing an ambiguous prefix registers a second commit sharing the prefix.
- `gitwrap_test.go` (integration, real git): a subtest in `TestRepositoryProbes` pins `commitsNamedBy`: a 9-digit prefix of HEAD answers HEAD's full SHA alone, the prefix of a committed blob's id answers no commit, and a prefix naming no object answers empty with no error.
- `recordbatch_test.go`, extending the tests that cover each path: a 9-digit abbreviation of HEAD, in lowercase and in uppercase, records the batch with the digest's `HeadSHA` and `CardSHAs` holding the full 40-digit lowercase SHA;
  in `TestRecordBatch_ParentMovedHead`, an abbreviation a clean parent merge sits above is accepted with the moved-HEAD warning naming the full SHA;
  an abbreviation naming no commit and one naming two commits are each refused with `ErrHeadSHAUnresolved` and the `git rev-parse HEAD` way forward, no reset named, the batch non-terminal and the report in place;
  a batch with a correctness finding fails with the full SHA in the failed digest when `head_sha` abbreviates HEAD, and with an empty digest head and no `ErrHeadSHAUnresolved` when it names no commit;
  in `TestRecordBatch_HeadSHAMismatchErrors`, a non-hex `head_sha` (such as `main`) still meets the mismatch refusal.

Docs: `doc.go`'s head cross-check paragraph states that `head_sha` may be an abbreviation of 4 to 39 hex digits, resolved to the one commit it names before any comparison, and that an abbreviation naming no commit or several is refused.
`refusal-spec.md` gains a webster row, `report head_sha unresolved`: record-batch finds the report's `head_sha` abbreviates no commit or more than one, correctness halt, way forward: have the fork rewrite `head_sha` as the output of `git rev-parse HEAD`, then re-run the verb (`PATTERN-refusal-way-forward`).

**ImpactSummary:** `Git` gains one method, implemented by `realGit` and the test `fakeGit`; `RecordBatch` records full SHAs where it recorded the report's raw value, and a short-hex `head_sha` that named no commit moves from the mismatch refusal to the new one.

**Commit:** 2: webster record-batch: resolve an abbreviated head_sha to the full commit SHA (#456)
