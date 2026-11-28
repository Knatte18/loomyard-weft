```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, find the test row that fails when that behaviour breaks.

**Per behaviour, the test that pins it**

1. A missing absolute local path on push, fetch or ls-remote is a transport failure: `transport_test.go:63`, plus the real-git loop at `transport_integration_test.go:63-76`. Pinned.
2. A `file://` prefix is stripped before the stat: none. Row `transport_test.go:64` uses a missing path, so it passes with or without `TrimPrefix`. Without the strip, `file:///…/removed.git` is not absolute, gets joined under `existing` and is still missing, so the result is still true.
3. A relative path is read against `GitError.Dir`: none. Row `transport_test.go:65` passes even if the join is dropped, because `Lstat("removed.git")` against the test process's directory also misses. No row has a relative path that exists under `Dir` and wants false. No real-git row uses a relative URL either.
4. The clone form `repository '<path>' does not exist`: `transport_test.go:66`, plus real git at `transport_integration_test.go:67`. Pinned.
5. An existing directory holding no repository stays a refusal: `transport_test.go:67`, plus real git at `transport_integration_test.go:78-87`. Pinned.
6. A local subcommand naming a missing path is not transport: `transport_test.go:68`. Pinned.
7. An unconfigured remote name: none. Real git prints `fatal: 'origin' does not appear to be a git repository`, which the classifier now stats as `<dir>/origin` and so reads as transport. No row pins either verdict.
8. A dangling symlink counts as existing, because the code uses `Lstat`: none. No row pins either verdict.
9. Reconcile over an origin missing on disk sets `Unreachable` with no error: `originreconcile_integration_test.go`, row "an origin missing on disk" (diff 002:48), with the new `report.Unreachable` assertion. Pinned.
10. Reconcile over an origin that is no repository is still an error and not `Unreachable`: the same test's row at diff 002:49, plus the subtest at diff 002:74. Pinned.
11. A push to a removed local origin writes one `transport` record and one mail: `pushclasses_integration_test.go`, row `ARemovedLocalOrigin` (diff 002:120). Pinned.
12. A push to a non-repository directory is still `other`: `APushFailingForNoNamedReasonWritesAnOtherRecordAndMails`, `pushclasses_integration_test.go:757-770`. Pinned.
13. The rewritten fixtures still check what their names say:
    - add `OriginNotARepository` (`add_remoteleftover_integration_test.go:105`): yes.
    - cleanupcli non-repository origin (`cleanup_integration_test.go:265-275`): yes.
    - pushanchored "rejected push with a failing fetch" (diff 002:85-99): yes.
14. The other callers (add retry, executor, pull, pushanchored, shedtransient) inherit the new classification. Only the classifier's unit rows cover them, which is enough at the classifier boundary. No finding.

**Findings**

- LOW, design, `internal/gitexec/transport_test.go:64`: the `file://` row cannot catch a dropped `TrimPrefix`; the missing path makes it true either way.
  - Fix: add a row with `file://` and an existing non-repository directory, wanting false.
  - Evidence: `transport.go:84-89`, `transport_test.go:64`.
- LOW, design, `internal/gitexec/transport_test.go:65`: the relative-path row cannot catch a dropped join against `Dir`; the relative `Lstat` misses from the test's own directory too.
  - Fix: add a row whose relative name exists under `Dir` but holds no repository, wanting false.
  - Evidence: `transport.go:85-87`, `transport_test.go:65`.
- MEDIUM, design, `internal/gitexec/transport.go:79-90`: no test pins the verdict for an unconfigured remote name, whose real stderr the new pattern matches and now classes as transport.
  - Evidence: real git 2.53.0 probe, `git ls-remote origin` in a repository with no origin printed `fatal: 'origin' does not appear to be a git repository`.
  - Evidence: `recordslayout.go:41` says a repository with no origin remote is a supported state.
  - Fix: decide the verdict, then pin it with a hand-built row and a real-git row.
- LOW, design, `internal/gitexec/transport.go:88`: no row pins how a dangling symlink, such as a link into an unmounted share, is classed under `Lstat`. The doc's "unmounted origin directory" claim covers that case. Evidence: `doc.go` diff 003:9-10, `transport.go:88`.

**Tool calls**

1. `sed` of `add_remoteleftover_integration_test.go:95-112`, `pushclasses_integration_test.go:735-775` and `cleanup_integration_test.go:255-268`: the manifest held only the diff hunks, not the rewritten fixtures' row and test names.
