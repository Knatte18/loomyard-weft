```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens on the `hardenLiveArgs` diff. For each behaviour the diff adds or changes, I name the test row that would fail if it broke.

**Each behaviour and the test that pins it:**
- **A bare entry gets a `./` prefix:** `internal/a` must come back as `./internal/a`. Pinned by `TestHardenLiveArgs`, in both subtests ("the live call names its key and the packages as directories go resolves…" and "the copies call carries --copies before the packages"), at `internal/loomcli/hardenwiring_test.go:44-59`. The expected values are literals, and the summary says this row fails on the merge base.
- **An entry that already starts with `./` passes unchanged:** `./internal/b` must come back as `./internal/b`. The same two subtests pin it with literal expected values. A change that adds the prefix every time would produce `././internal/b` and fail.
- **An entry of `.`, `..` or `../x` passes unchanged:** no row of its own. The `./internal/b` row already pins the same branch (`build.IsLocalImport` true, no prefix), so the extra inputs would be rows inside the stdlib predicate's own contract. No finding.
- **Packages still follow the `--copies` arguments in the same order:** the copies subtest pins it with a literal.
- **The live call still has no `--copies` when copies is 0:** the first subtest pins it.
- The mutation report shows 3 of 3 mutants killed and no survivors. The reach report shows `TestHardenLiveArgs` reaches `hardenLiveArgs`.

**Findings:** none. Every behaviour the diff adds or changes has a row with an independent literal expected value that would fail if the behaviour broke. The bare-prefix case is the behaviour the decision record asks a test to pin ("A test pins that a bare `internal/x` reaches `gate test` as `./internal/x`"), and the test pins it with `internal/a` → `./internal/a`.

Outside my lens, for the code-correctness lens or the handler: absolute paths and the empty entry are not covered. Whether they are valid inputs at all is a correctness and scope question, not a coverage gap here.

Tool calls: none.
