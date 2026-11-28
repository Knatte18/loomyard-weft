```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the round-1 change set (`e938587a9`, `d3bad1e1e`, `cae2038c9`).

1. **LOW**: `internal/landingshed/doc.go`, the added line "A landing known to need GitHub always merges the upstream tip there."
   - "always" claims more than the code holds. `parentMergeTip` takes the local tip when `deps.PushSkipped` is set or the parent reads `ParentNoUpstream`, whatever `takeOwedLocal` is. So `fz.needsGitHubOnDisk()` being true does not guarantee the upstream tip.
   - The twin sentence in the `gateUnlocked` doc (`finalize.go`) is scoped as an exception to the owed-ahead case ("is the exception and merges the upstream tip"). The `doc.go` wording lacks that scope.
   - A sound wording would be: "A landing known to need GitHub merges the upstream tip of an owed-ahead parent too."

I found nothing else stale:
- The `ParentAheadByOwed` doc in `deps.go`, the `Finalize.Call` doc, the `needsGitHubOnDisk` doc and the `Finalize` and `Parent-Align` paragraphs of `docs/overview.md` still agree with the changed code.
- The rewritten `parentMergeTip` doc agrees with its body.

Tool calls:
1. Grep of `.md` and `.go` files for owed-landing and lock-free wording, to find twins outside the files already read.
2. `docs/overview.md` lines 1355-1425, which were not in context.
3. `internal/landingshed/finalize.go` lines 205-240 (the `Finalize.Call` doc), since the context's read began at line 300.
