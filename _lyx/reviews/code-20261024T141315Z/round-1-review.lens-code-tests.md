```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it:**

The range `93f1c8942..HEAD` adds no `Test`, `Benchmark`, `Fuzz` or `Example` function and removes none. A `git diff` filtered for `^[-+]func Test` returned nothing. The test-file edits change comments, `//testtiming:keep` reasons and `t.Errorf` message text, plus a reflow of two `configstrictness_test.go` comments into one sentence per line. No assertion condition, pinned set or table row changes.

- **Cited-ID renames in guard-failure messages.**
  - The guards are in `cmd/lyx/agentfiledissues_test.go`, `checkedcall_test.go`, `configstrictness_test.go`, `destructiveguard_test.go`, `norecoverydeny_test.go`, `removestrand_test.go`, `verifiedtree_test.go`, `tiertags_test.go`, `constraintchokepoint_test.go`, `spawnrole_test.go` and `internal/planparser/shape_enforcement_test.go`.
  - Each guard's own test pins the behaviour. Only the message text changed, and no test reads a message's cited ID. Nothing uncovered and nothing redundant.
- **`//testtiming:keep` reasons.**
  - Those in `contracts/stencils/parentdirective_test.go` and `roleskills_test.go`, and the one in `norecoverydeny_test.go`, now name the survivor `PATTERN-parent-directive` or `PATTERN-refusal-way-forward`.
  - Each reason still names what its test pins that no other test does, so the keep contract holds.
- **`cmd/testtiming/redundancy.go`, the report footer.**
  - The footer now points at "`pattern/PATTERN-test-speed.md`'s test economy section".
  - Gap (LOW, see below): I did not see a test pinning this string, so I could not confirm one covers it. The footer is output text, not logic.
- **`internal/lyxcwd/docslink_test.go`.** Comment only. The test is unchanged and still pins docs link and anchor integrity, which survives as a test after the `markdown-link-integrity` entry was dropped.
- **Moved rules.**
  - The record says no test or scan was removed, and the diff bears that out.
  - The guards for moved rules still exist: `TestMutationRecord_FabricengineProductionSource`, the `constraintchokepoint` test, the `spawnrole` test, `sandbox_coverage_test.go`, and the `panebin`, `completionsignal` and `modelchoicemapping` enforcement tests.
- **Test economy:** no duplicated test, no per-helper test, no review-fix test, no new assertion on stencil prose.

**Findings**

1. **LOW / consistency** — `cmd/testtiming/redundancy.go` (the footer line `b.WriteString("Only the removable set may be deleted together; the reading rules are in ...")`). I could not confirm that any test asserts this string, so the retarget from `PATTERN-test-economy.md` to `PATTERN-test-speed.md` may be unpinned. It matters only if such an assertion exists and still cites the old path. Weak: I made no search, so treat it as a check for the handler, not a defect.

No other findings.

Tool calls (1 of 6):
1. `git diff 93f1c8942..HEAD` on `impactset/guards_test.go`, `impactset/guards.go` and `shellguard/rows_integration_test.go`, plus a grep for added or removed test functions across all `*_test.go` files. My context did not show whether any test function changed in the range. The three named files had no change in it; they changed only in the earlier parent range.
