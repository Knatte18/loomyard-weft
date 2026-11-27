```yaml
lens: fix-code-correctness
outcome: reported
```

Task: checking the correctness of the code the fix changed (fix-code-correctness lens).

No findings.

- **Changed behaviour:** none. The commits for F2 and F3 change comments, package doc and stencil prose only. No signature or caller changed.
- **New test row:** `FailedBuildSkewTransient_BuildSkew` has state failed, `buildSkew` and `transient` set, and `interrupted` false. It skips the blocked and interrupted cases in `ClassifyHalt` and hits the build-skew case, returning `(AnomalyBuildSkew, true)`. With the two failed cases swapped it would return `transient-stop`, so the row catches the defect F4 names.
- **Reworded claims:** they match the code shown. `shedverbs/loop.go` re-steps a transient envelope when `!restepped`. `decideDrive` gives `git-transport` and `github-api` re-steps with waits and sends other non-empty classes to the tackler, unless an earlier general rule (build changed, resume returned) takes the stop first.

Further tool calls: none.
