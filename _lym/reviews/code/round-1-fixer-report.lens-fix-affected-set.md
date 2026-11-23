```yaml
lens: fix-affected-set
outcome: reported
```

**Task:** the fix-affected-set lens, re-running each set-premise finding's search to see whether the fix covers every member.

**Findings**

1. **LOW, F1 sibling.** `internal/gitrepo/blobread.go:169` (`FilesInDirAtRevision`) and `:197` (`FilesUnderDirAtRevision`) still filter with `IsRegular()`.
   - Evidence: go-git's `IsRegular` is false for `filemode.Executable`, the premise of F1. Both listings therefore still omit `100755` blobs.
   - Their doc comments say "regular files", so this is arguably intended behaviour. Executables are still silently missing from these reads.
   - Callers decide whether it matters. None were checked, because this was outside the fix's reach.
   - Not a regression from the fix.

2. **LOW, F2 sibling.** Other `Scoper ==` / `!=` literal branches remain in `internal/gatecli/scope.go:167,180,186`, `tiers.go:170,173` and `run.go:285`. `redfirst.go` now has none.
   - These branches were already in the code.
   - F2 scoped its fix to the three new red-first sites in `redfirst.go`, so they are not new, and the fix covered every site the finding named.
   - The `RedFirstMode` table cannot replace them, because they are not red-first behaviour.

3. **No finding, F12.** Another `<package>.<Name>` mention at `internal/gatecli/redfirst.go:575` is `testLabels`, the Go path's own function, so it is correct as written. The `doc.go` exit-contract line is the only one the dotnet path made wrong.

4. **No finding, F3/F4/F11 call sites.** `judgeDotnetRedFirst` has one caller, `redfirstdotnet.go:131`, and it handles `noTestRan`.

**Unchecked:** none. F5–F10 and F13 name a single site or a test row, not a set.

**Tool calls**
- One `grep -rn "IsRegular()" internal`, for the sibling sites of F1's premise (my context held only `blobread.go`).
- One `grep -rn 'Scoper [=!]= "' internal`, for the remaining scoper-literal sites for F2.
- One `grep -rn "judgeDotnetRedFirst\|<package>.<Name>" internal docs`, for the callers of the changed judge and the other mentions of the `<package>.<Name>` label for F3/F12. These sat in the same Bash call as the two greps above.
