```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff changes, find the test row that would fail if that behaviour broke.

The diff touches only test files. The behaviours it changes are each row's starting state, now built per row. So for each row, the question is whether it still fails when the behaviour its name states breaks.

**Per behaviour, the test that pins it:**
- **Computed, nested-module, Markdown-only, inert-only (go/none/dotnet) and untracked-beside-inert rows (Scopes):** pinned by their own rows. Each edits a fresh hub and asserts exact tier args, so the base start state is reproduced. Covered.
- **Shared-build dotnet rows, `--base`, `//lym:watch` computed and dotnet computed/project-file rows:** pinned by their own rows. They are unchanged apart from the fixture. Covered.
- **`--card` rows (packages per module, dependent guards, `//lym:only`, dotnet glyph, unreached project, watch with no parent branch, vet, vet failure, timeout, unknown id ×3, no plan, `--main-packages` ×4, covered tier not run):** pinned by their own rows. Removing the "dirtying note" edits is safe because a fresh hub holds no green record, so nothing carries. Covered.
- **`--gate` all/diff/targets/unknown key, misplaced guard, prime with no parent, none scoper:** pinned by their own rows. Covered.
- **"an all run after a green diff run…":** pinned. The base removed the green record first and a fresh hub has none. Covered.
- **"a changed tier command, a different tree and -- -count=1 each break the carry":** pinned. The new priming `--all` run is asserted to run `./...` whole (`scopes_integration_test.go`, part 2 l.564), so the later carry checks start from a real green entry. Covered.
- **"--card carries a vet run…", "a tier whose command … commits … records nothing", "the green entry of a run that held tests back…":** pinned by their own rows. Covered.
- **"a diff run diffs from the newest whole-green ancestor, else from the parent merge base":** pinned. It commits before the `--all` run, so `green` differs from `mergeBase` and the two derivation checks can tell them apart. Covered.
- **"a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor":** **none**. See finding 1.
- **"tiers on one base share one derivation line…":** pinned. It commits `Seen=3` before recording `green`, so `green` differs from `mergeBase` for the split check. Covered.
- **Dotnet narrowed pass and the inert-carry `--all` rows ("carries over a committed inert-only change", "runs after a committed non-inert change", "on a dirty tree runs whole", "carry: false", "changed inert_paths"):** pinned. Each now primes its own whole-green `--all` run before its commit, so the "whole ancestor is green" precondition really holds. Covered.
- **None-scoper Markdown-only row and the RunGate observer row:** pinned by their own rows. Covered.
- **Scenario carry rows (`--gate` carry, `--copies` records nothing, `--all` shared-build carry, `--guards` groups):** pinned. Each cleans its own fresh hub, and a fresh hub holds no green record, which matches the base's removal. Covered.
- **"a dirty worktree carries nothing and writes nothing" (Scenario):** pinned. A fresh hub starts dirty, as the base did. Covered.
- **Shared-build, env-module, slot, status, `--wait`, `-C`, real-go, lint, crash table, stop-on-fail, timeout, orphan, signal, prebuild, standalone and guard-scan rows:** pinned by their own rows over a fresh hub or standalone worktree. "a real go test tier in the nested module" now creates `nested` itself. Covered.
- **Refusals table:** pinned. Each row's `setup` breaks its own hub, and `wantError(f)` derives the expected paths from the same `f`. "a slot directory that cannot be created" runs `MkdirAll` on `pool.Dir` before renaming it, so it works on a fresh hub. Covered.

**Findings**

1. **BLOCKING · design** · `internal/gatecli/scopes_integration_test.go`, row "a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor" (diff part 2, l.730–744).
   - **Summary:** on a fresh hub the whole-green ancestor is the merge base itself, so the assertion passes even if the dirty-tree rule is broken.
   - **What happens at HEAD:** the row runs `--all` on a pair that still stands on `f.mergeBase`, so the green whole entry is written for `mergeBase`. It then commits `lib=9`, dirties the tree, and asserts `wantDerivation(…, f.mergeBase, "tier0")`. A gate that wrongly used the newest whole-green ancestor would print the same `mergeBase`.
   - **What the base did:** the row started after the previous row's commits, so the pair's HEAD (`lib=8`) differed from the merge base. Its `--all` run recorded green at that HEAD, and the check could tell "merge base" from "green ancestor" apart (base `scopes_integration_test.go:1062`, with the previous row's commits shown in diff part 2, l.702–728).
   - **Fix:** commit a change on the pair before the `--all` run, so the green ancestor is not the merge base.

No tool calls made: the inherited context held everything this lens needed.
