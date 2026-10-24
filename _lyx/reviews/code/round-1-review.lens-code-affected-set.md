```yaml
lens: code-affected-set
outcome: reported
```

**List: per change, the set found, covered or not.**

1. **`tierRun.vetStep`, `scopeVet`, `runLabel`, `tierNumbers` (run.go, tiers.go): covered.**
   - Set: the closing-line label (`printFailedRuns`) and the `execution.tierList` that `report.go:114` reads from `tierNumbers`.
   - The run loop at `run.go:140` sends a vet run to `runListed`, then `runGreenAware`. So a vet run takes the slot, summary line and carry paths like a listed tier.
   - The new integration test pins `failed runs: vet tier1`.

2. **Summary line `gate test: tier1 vet … exit N` (consumers `gateslot.FailedTier`, `tierSummaryPattern`): covered, with one observation.**
   - `tierSummaryPattern` (`runlog.go:28`) matches `tier(\d+) .* discovered …`, so a vet scope form still parses.
   - `landingshed/verifygate.go:96` takes a failed vet line as "tier N failed" and writes `failure.Tier = 1`. This is not a defect. Nothing in that path tells a vet failure from a listed-tier failure, and `--card` scopes are not landing gates. The scope form `vet` in the summary line carries the distinction.

3. **Green record (`green.go:283` `subtract`, `green.go:337` `recordPass`): covered.**
   - The vet `Tier` has an empty `Dir` and `AllTargets`. In `recordPass`, the `{targets}` branch for `slices.Equal(inv.targets, Fields(""))` is false, so it falls to `expandPackages` and records packages.
   - `expandPackages` reads `-tags` from the command with `buildTagsPattern`, so a vet pass records packages under `tierN`. No stale whole-entry misuse.
   - The vet command string differs from the tier command, so `tierFingerprint` differs.

4. **`RunRecord.ScopeForm` readers: covered.**
   - The only other reader is `report.go:114`, which prints the scope form and is generic. No switch on scope form exists elsewhere.
   - The `GuardStep` flag is not set for vet runs. That is consistent with the decision record, and nothing else reads it.

5. **doc.go and docs/overview.md: covered.**
   - Both are updated and consistent. The guard paragraph's "after the last listed tier and vet run released" is updated too.
   - `test_integration_test.go:175` `realGo` comment is updated.

6. **`cli.go:55` `testCmd.Long` help text: not covered.**
   - Severity: LOW. Class: consistency. Location: `internal/gatecli/cli.go:55-75`.
   - The `--card <id>` / scope text lists no vet step. The user-visible verb help says nothing of the new vet run, while the package doc and overview describe it.
   - The latest commit title says it keeps the verb's Long "under its cap", so this may be deliberate. The help still lacks any mention of a vet run a `--card` call now plans and names in its closing line.
   - Fix: add one short line to the Long, such as "A --card scope under the go scoper also vets the card's packages under the other tier tags", within the cap.

7. **Existing `--card` tests that run under the stand-in go (`scopes_integration_test.go:204`, `:262`, `:298`, `:357`): not checked.**
   - I could not tell whether these fixtures hold a file that only a tier tag adds to a card package. If they do, the new vet run would add a `vet -tags` stand-in invocation their `wantArgs` do not expect.
   - The diff changes none of them. The new subtests use `edit` with a cleanup restore, so the added files do not leak to later subtests.
   - Not a finding.

**Findings**

- LOW / consistency / `internal/gatecli/cli.go:55-75`: the `lyx gate test` Long help omits the new `--card` vet step. Evidence: the Long text above, which names only the guard step for narrower scopes.

**Tool calls (4):**
1. grep for `ScopeForm`/`GuardStep` readers, `runlog.go` 170-200 and `subtract`/`recordPass` in `green.go`: my context lacked the consumers outside the changed files.
2. grep of `--card` tests, the stand-in go in `test_integration_test.go`, and the `docs/overview.md` diff: to find existing tests the vet run could disturb, and to check the doc.
3. `tierSummaryPattern`, `FailedTier` callers, the `edit` helper and `Long:` locations: to check the line parse and the test cleanup.
4. `verifygate.go:85-115` and `cli.go:55-80`: to see what a vet failure's tier number feeds and what the verb help says.
