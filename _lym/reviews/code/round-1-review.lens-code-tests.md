```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, name the test that pins it, or none.

**Per behaviour**

| Behaviour | Pinning test |
|---|---|
| Setting on: asks once, naming prefix, slug, PR URL, "branch task into main" and `lym creel post pr-strand`, and sets `ParentReviewAsked` | `TestPRReview_Session/ParentReviewOnAsksTheParentOnceNamingTheSlugPullRequestBranchesAndSession` (`prreview_session_test.go`) |
| Record already asked: no second ask | `.../ParentReviewOnWithTheRecordAlreadyAskedAsksNone` |
| Setting off: no ask, record untouched | `.../ParentReviewOffAsksNone` |
| Unreadable setting: logged, no ask | `.../AnUnreadableParentReviewSettingAsksNone` |
| Post error: logged, record stays unasked, so the next entry asks again | `.../AFailedParentAskLeavesTheRecordUnaskedForTheNextEntry` |
| No orch recorded (`posted` false): record stays unasked | `.../AParentAskNoOrchTakesLeavesTheRecordUnasked` |
| `ParentReviewAsked` survives a JSON write and read (key `parent_review_asked`) | Indirect: `assertAsks` reads the record back through `pullrequest.ReadRecord` |
| `pr_review_ask_parent` is a loom key that a layer can override | `config_test.go` row "pr_review_ask_parent overridden" |
| The key is classified, so the local layer may hold it | `TestModules_EveryTemplateKeyClassified`, which I know only from `PATTERN-config-layers.md`; the test file itself is not in my context |
| Setting defaults to off | Not pinned by a dedicated row. The template says `false` and the Go zero value is `false`, so a default of `true` would only be caught if the existing defaults test compares the whole `Config`. |
| New `Deps` seams are filled in the loom wiring | `internal/loomcli/wiring_seams_test.go` and `internal/battencli/wire_seams_test.go` list them as intentionally nil, with a reason. See F1. |
| Orch role and reviewer stencil prose | Exempt as static prose. |

**Findings**

- **F1, MEDIUM, design.** Location: `internal/loomcli/landingdeps.go:419-426`. The closures behind `ParentReviewWanted` and `AskParent` have no test.
  - `ParentReviewWanted` reads `loomengine.LoadConfig(fabricengine.ConfigDirs(l), "loom")`, and `AskParent` calls `orchcli.NotifyParent(l, loomProgram, line)`.
  - If the per-task `run_settings` layer were not read, or `AskParent` were wired to a different notifier, the shed tests would still pass, since they use fakes.
  - The seam-test rewrites only list these seams as intentionally nil, because run.go fills them; they never assert the filled value.
  - The feature's core promise, "a per-task override turns it on", is untested end to end.
  - Evidence: `landingdeps.go:419-426`, `wiring_seams_test.go:47-48`, `wire_seams_test.go:107-108`.
- **F2, LOW, scope.** Location: `internal/landingshed/prreview.go:337-340`. The nil guard `ParentReviewWanted == nil || AskParent == nil` ("nil reads as off", per the `deps.go` comment) has no row.
  - `prReviewTestDeps` and `envkit.LandingDeps` always set both seams.
  - A nil seam is a tolerated default, not an internal misuse, so a row would pin it.
- **F3, LOW, scope.** Location: `prreview.go:359`. The `updateRecord` failure after a successful post, which returns an error, has no row.
  - The docs and comment promise "a failed write of the ask into the record is an error".
  - The other error rows all stay `Done`.
- **F4, LOW, scope.** Location: `internal/landingshed/doc.go` and `prreview.go:323`. Two claims have no row.
  - "A review started afresh writes a new record and so asks again."
  - The ask happens only on the open path after `ensureSession`; the closed, merged and `ReviewEnded` paths do not ask.
  - Every row uses the open path with `Create`.

**Tool calls:** none. The manifest and diff already in my context were enough.
