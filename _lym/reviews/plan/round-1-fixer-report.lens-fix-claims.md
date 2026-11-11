```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

no findings

What I checked:
- **Card 1's re-read claim holds.** The Lost report and the shared re-read match `internal/boardengine/ownerset.go:141-160`. `CompetingWriteLosesToOrigins` at `internal/boardengine/ownerset_integration_test.go:453` asserts Lost to `box` with origin's owner standing. `Recorded` is built from `written` in that same loop, so `ReplaceSetSelectsAfterThePull`'s `Recorded` assertion does pin that replace-selected slugs enter it.
- **Card 1's three doc sentences are quoted correctly** from `ownerset.go:43`, `ownerset.go:72` and `doc.go:183`.
- **Card 4 step 7's error-text claim holds.** Some `SetOwners` errors name a way forward and others don't: `ownerset.go:288` names `lym board sync`, while `:122` and `:278` don't. So "names its own way forward where SetOwners gives one" is accurate, and the re-run as the way-forward field doesn't depend on testing the error.
- **The new rows' expected names follow from step 5's rule and `CheckName`** (`internal/machinename/machinename.go:90`):
  - `OwnFormerNamePassesTheCollisionCheck`: `BAD` is invalid because it is upper-case, so no old name is added. Host `box` is appended, then `osl46` moves to the end, giving `["neve","box","osl46"]`.
  - `EmptyEntryDropIsReported`: `["","box"]` plus `box` already present, one empty entry dropped, then `osl46` appended, giving `["box","osl46"]`.
  - `NoFileTakesTheHostnameNameAsOld`: gives `["osl-1046","osl46"]`.
- **`CwdContainerDirectoryIsSwept` holds.** In a bare temp dir, `lymcwd`'s `gitWorktreeRoot` returns `ErrNotAGitRepo` (`internal/lymcwd/lymcwd.go:152-162`), so `hubOfDir` falls back to the shortname read. `ReadShortname` and `WriteShortname` both take the directory that holds `.lym-shortname`, which matches `ReadShortname(BoltDir(hubPath))` at `internal/fabricengine/hubworkspace.go:97`. `WriteShortname` doesn't create that directory, so the test makes `BoltDir` first. That is a setup step, not a false claim.

Tool calls:
- Read `CheckName`, the shortname helper signatures and `gitWorktreeRoot`, which my context didn't show, to check the expected names and the container-directory row.
- Read `ownerset.go:118-124` and `:274-290`, which my context didn't show, to check the error way-forward claim.
- Read `shortnamebinding.go` and the shortname call sites, which my context didn't show, to check which directory the helpers take.
