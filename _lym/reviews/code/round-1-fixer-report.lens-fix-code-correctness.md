```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fix's changed code and tests (fix-code-correctness).

I traced these by hand and found no wrong result, no broken caller and no added test that would pass with the defect in place:
- `ignoredWork`, `ignoredPaths` and `mayHoldWork`
- `lymWrittenInPrime`
- the Bolt HEAD count
- the F3 and F9 error paths
- `RefuseHubWithoutBolt`
- the new test rows

One finding:

- **LOW**: `tools/sandbox/SANDBOX-CORE-SUITE.md`, S6, the sentence "That clone left its module-config commit unpushed, so a plain `--reset` is refused."
  - The sentence states this unconditionally.
  - The same section says the later `unwire --apply` step "commits and pushes that deletion to the shared `lym-test-weft` remote". If that push carries the records branch, it carries the earlier module-config commit with it. A leftover hub on which `unwire` ran would then not be refused for that commit.
  - `--reset --discard-work` still works in both cases, so only the stated reason is too broad.
  - Evidence is the suite's own text. I did not read `unwire`'s push code.

Outside my lens: skipping `.vscode/settings.json` and `.claude/settings.local.json` means `--reset` removes any operator-added keys in them without naming them. The review's fix asks for this skip.

Further tool calls: none.
