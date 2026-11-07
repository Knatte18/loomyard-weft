```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it (or none):**

| Behaviour | Pinning test |
|---|---|
| Different keys build at once, each exactly once, under parallel first callers | `TestCopyRegistered_KeysBuildConcurrentlyOnceEach` (`template_test.go:61`). The rendezvous fails if the builds queue. The count check fails if a key builds twice. |
| Map lock not held across a build | The same test. A held map lock would queue the builds and trip the rendezvous. |
| Copy holds the build's commit and pairs, and the binding fold keeps the build commit | `TestCopyRegistered_CopyHoldsTheBuildAndIsolatesWrites` (`template_test.go:101`). It checks the file, the log subject, the upstream-ahead count and both pairs' worktrees. |
| A write to the copy leaves the template unchanged | The same test. It compares the fingerprint and checks that a later copy lacks the write. |
| Failed build: the first caller gets the build's error naming the key, later callers name the key, and there is no rebuild | `TestCopyRegistered_FailedBuildFailsEveryCallerNamingTheKey` (`template_test.go:183`), via the `Fatal` row. Every row also asserts `builds == 1`. |
| Refusal of `Cleanup`, `Setenv`, `TempDir`, `Chdir` and `ArtifactDir`, each naming the key | Five rows of the same table (`template_test.go:163-167`). |
| Board ahead of the bare tip fails | The `BoardAheadOfTheBare` row. |
| Tip read from the bare, not the tracking ref; a commit pushed from a clone inside the template root fails | The `BarePushedFromElsewhere` row. The board's tracking ref stays equal to HEAD there, so only a read from the bare fails it. |
| Two registrations with one key fail the second naming the key | `TestCopyRegistered_SecondRegistrationOfAKeyFailsNamingIt`. |
| The same registration used twice is accepted | The two copy tests above, which use one registration repeatedly and in parallel. |
| Shapes build through registrations: `CopyHub`, `SharedHub`, nil `Build`, relocation and rebase unchanged | `TestCopyHub` and `TestSharedHub` in `hub_test.go`. `TestCopyHub` has the one-line diff to `shapeRegistration`. |
| Skip refusal (`Skip`, `Skipf`, `SkipNow` fail the build) | **None.** |
| Non-fatal failure recording (`Error`, `Errorf`, `Fail`, `FailNow`, `Fatal`; `Failed()`) fails the build | **None.** The only recording row is `Fatalf`, and the tip check reaches the TB only through `Fatalf`. |
| Shape outside `Shapes` fails, with the new message naming `Registration` | **None** (grep for `outside Shapes` in `*_test.go` found nothing). |
| A registration key equal to a shape name counts as a shared key | **None.** The summary claims it. |
| A poisoned registered template fails `CopyRegistered` | No registered-template row. `copyTemplate` is shared, and `TestSharedHub` already covers `CopyHub` after poisoning (`hub_test.go:688`). |

**Findings:**

1. **MEDIUM, design.** Skip refusal has no test row. Location: `template.go:297-307`, `Skip`, `Skipf` and `SkipNow` each calling `refuse`. The doc, the `Registration.Build` comment, `PATTERN-test-isolation.md` and the summary all state it as a behaviour. No `failingBuilds` row at `template_test.go:161-178` calls any skip method. If a skip method stopped refusing, the first caller would be marked skipped from the build's goroutine and nothing would fail. Evidence: the grep for `Skip` in `internal/hubforge/*_test.go` returned nothing.

2. **LOW, design.** The build's recording of non-fatal failures has no row. Location: `template.go:266-295`, `Error`, `Errorf`, `Fail`, `FailNow`, `Fatal` and `Failed`. A build that calls `tb.Errorf` or `tb.Error` and continues must still fail the build. Only the `Fatalf` path is exercised, so a regression to forwarding `Errorf` to the real TB would pass. Fix by adding one `Errorf` row to `failingBuilds`.

3. **LOW, design.** The shape-outside-`Shapes` failure has no test. Location: `template.go:91-99`, `shapeRegistration`. The diff rewrote this path and its message. The old check had no test either, so this is a gap rather than a regression. One row in `TestCopyHub` or `TestSharedHub` would pin it.

4. **LOW, scope.** The summary's claim that a registration key equal to a shape name counts as a shared key has no row (`summary.md`, "Design decisions"). The mechanism is the one the shared-key test pins, so this is only a missing instance.

**Test economy:** no duplicate tests and no assertions on static prose. `TestCopyRegistered_*` does not repeat `TestCopyHub`, which covers relocation of shapes while the new tests cover builds and failures.

**Tool calls (1):**
- Grep in `internal/hubforge/*_test.go` for `outside Shapes`, the old and new copy error prefixes, `failed to build` and `Skip`. My context lacked the rest of `hub_test.go`, and it settled whether a test asserts the changed messages or the skip refusal.
