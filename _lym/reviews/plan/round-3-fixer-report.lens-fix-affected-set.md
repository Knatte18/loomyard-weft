```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran each finding's own search to check that the fix covers every member of its set.

Unchecked findings: none.

No findings.

- **F1** (fields of `testMapRun` that only deleted code reads): I re-read the struct's fields and checked each field `prepare` sets for a reader that survives card 1. `fabric`, `hubEnv`, `toolchain`, `moduleRel` and `location` each keep a reader outside the deleted code (`testmappackage.go:910-912`, `testmaptree.go`, `testmappackage.go:109`/`591`). `selector`, `dirty`, `plan`, `selfCheckMismatches` and `auditMismatches` were already in the drop list. `lymBuilt`, `lymBin`, `lymErr` and `parallel` are dropped by card 3. `inert` was the only field missing, and the fix adds it at card 1:106 and its assignment at card 1:113.
- **F2** (the `runtime.NumCPU()` fallback): grep of the plan finds no remaining `NumCPU` or "`Parallel` of 0" text, so both the bullet at card 3:58 and the test row at card 3:94 were changed. In the code, the only map-run fallback is `testmappackage.go:606` (`r.parallel` in `takeSlot`). Card 3 replaces `parallel` and does not carry that fallback into `mapSideBySide`. The fallbacks in `redundancy.go:433` and `redundantmutants.go:517` belong to `lym gate redundant`, which card 3 leaves alone.

Tool calls:
1. `sed` of the `testMapRun` struct in `internal/gatecli/testmap.go`: my context held only the grep for `inert`, not the full field list.
2. grep of `NumCPU` in gatecli's non-test files: my context lacked the code-side fallback sites.
3. grep of readers for `fabric`, `hubEnv`, `toolchain`, `moduleRel` and `location`: needed to confirm no other field becomes write-only.
