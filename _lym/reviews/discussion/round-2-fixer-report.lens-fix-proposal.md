```yaml
lens: fix-proposal
outcome: reported
```

I challenged the proposals the fixer carried out as written: F20's installer in `internal/fabriccli` that `internal/hubforge` calls, and F21's takeover-write half. I found three issues.

1. **MEDIUM — the unset-seam refusal test can't run alongside tests that build hubs.** Changed text: D13's installer paragraph, "`internal/hubforge` … calls it when it builds a hub", and the follow-up "a test of the unset-seam refusal clears the seam through the installer's restore".
   - The case: the installer writes process-global state, the same way `InstallPairFolderNamer` and `InstallWorkspaceContent` return a `restore` (`internal/fabricengine/hubworkspace.go:91`, `:115`). 189 test files in `internal/fabricengine` call `t.Parallel()`. Each of their hub builds re-runs the installer, so it can put the seams back mid-test after the refusal test cleared them, or the refusal test's clearing can unset them under a parallel test. Either way the criterion "A test shows a verb run with a required seam unset refuses" becomes order-dependent.
   - What's missing: the record doesn't say that test must stay out of `t.Parallel`, or that it needs a binary that builds no hubs, or that hubforge installs only once per binary.

2. **MEDIUM — hubforge's real seams make fabric's own tests drive shed and stencil code.** Changed text: "A fabric test that builds a hub through `internal/hubforge` runs with the installer's real seams, which D12 counts as the mandated fixture path."
   - The case: D12 exempts hubforge only for "reaching modules above fabric *to build a hub*". With the seams installed, every fabric tier1 test that runs a takeover, code-tip settle or warp forward on a hubforge-built hub calls shed's codec and adapter writes, or the stencil judge, while the verb runs. That isn't building a hub.
   - What it contradicts: D12's "a module fabric's tests drive sits above fabric", and D13's own "`internal/fabricengine` tests that flow's git effects as action-kind tests over bytes and paths, which call no codec or judge". The new sentence stretches D12's exemption without saying it does.

3. **LOW — fabric's takeover type has to stay in step with shed's, and nothing pins it.** Changed text: "fabric's own takeover entry type, whose JSON shape is today's".
   - The case: if shed later adds a field to `shedengine.Takeover`, fabric's copy won't declare it. `internal/state`'s merge (`keepUnknown`) then keeps the old takeover's value for that key under the new machine, subject and time, so the entry mixes a stale value with fresh ones.
   - What's missing: the same-bytes criterion only compares against today's code, and no check pins the two types' shapes to each other. The one-key patch carries this risk only because the proposal re-declares shed's shape in fabric.

Tool calls beyond my context:
- Counted the `t.Parallel()` files in `internal/fabricengine` tests and looked up the install functions' `restore` signature. My context didn't show whether fabric's tests run in parallel or how an install hands back its previous state.
