```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to a point in the decision record's Goal (code-scope lens).

**Goal points** (the decision record has no `Added after Discussion` entries):
- **G1:** read GitHub's plan-gated 403 on the rules read as "no rules apply". `_lym/discussion/decision-record.md:20`
- **G2:** the fallback ("or at least"): name the real cause and a way forward that works. `_lym/discussion/decision-record.md:21`

**Changes, and what each traces to:**
- **C1**, `internal/pullrequest/client.go`: `planGatedMessagePrefix`, `rulesFeatureUnavailable`, and the early return in `RequiresPullRequest`. Traces to G1. This is the read Finalize's `resolveWithParentRule` makes (`internal/landingshed/pullrequestmode.go:193`).
- **C2**, `internal/pullrequest/client.go`: the same early return in `RequiredChecks`. Traces to G1. G1's reason, "a repository without the rules feature has no rules", covers every read of the endpoint. `RequiredChecks` reads the same endpoint for `ReadMergeState` (`internal/pullrequest/mergestate.go:78`), so the two reads now agree.
- **C3**, `internal/pullrequest/client_test.go`: the `plan` and `forbidden` scripted branches and their rows. Forced by C1 and C2.
- **C4**, `internal/pullrequest/doc.go:26-28` and `internal/landingshed/doc.go:152`: the package docs state the new reading. Forced by C1 and C2, which made the old doc text incomplete.
- **C5**, `docs/research/single-repo/sandbox-3-reclone-2026-11-21/report.md:28`: the anchor fix `#the-first-hub-after-the-pull` → `#3-the-first-hub-after-the-pull`. It traces to no Goal point and no traced change forces it. The repository's docs-link gate forces it instead: `TestEnforcement_MarkdownLinks` (`internal/lymcwd/docslink_test.go:422`) fails on a missing anchor. The merge base's report links to an anchor its heading does not produce (the heading at `report.md:100` is `## 3. The first hub after the pull`).

**Which changes build each Goal point:**
- **G1:** C1 and C2, pinned by C3 and documented by C4.
- **G2:** none, and correctly so. G2 is a fallback that applies only if G1 is not done, and the diff does G1.

**Findings:**
1. NIT, class `scope`. `docs/research/single-repo/sandbox-3-reclone-2026-11-21/report.md:28`: the anchor fix is outside the Goal. The docs-link gate forces it, since the merge base carried a broken anchor that `TestEnforcement_MarkdownLinks` rejects. But `_lym/landing/summary.md` never mentions it, and no commit trailer explains it. Fix: keep the edit, and add one line to the change description naming it as a fix the gate forced.
   Evidence: commit `03603d1c` touches only that file; `internal/lymcwd/docslink_test.go:1,422`; report heading at `report.md:100`.

No other finding: no Goal point is left unbuilt, and every other change traces to G1 or is forced by a change that does.

Tool calls:
- `git show --stat 03603d1c` plus a grep for an anchor-checking gate: the context lacked whether the link fix was forced by a gate.
- grep of `internal/lymcwd/docslink_test.go` for its tests and anchor checks: to confirm that the gate fails on a missing anchor.
