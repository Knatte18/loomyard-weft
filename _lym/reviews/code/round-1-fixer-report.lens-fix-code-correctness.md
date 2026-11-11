```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

I traced each changed branch by hand against the code in context:

- **`sweepHub` (`internal/machinecli/set.go`):**
  - With `Recorded` empty and `Lost` `{"s2":"far"}`, the outcome is `outcomeKept` and `Lost` is populated.
  - `setName` only upgrades `outcomeNothing` to `outcomePushed`, so a kept hub stays kept even when the board was ahead.
  - `runSet` counts `len(hub.Lost) &gt; 0`, so the exit stays non-zero.
  - The old `Recorded &gt; 0` case still gives `outcomeSwept`, and a failed write still gives `outcomeFailed`.
  - Nothing else reads `outcomeNothing` in a way the new constant breaks.
- **`LostOnlyHubReportsKeptNotNothingToSweep`:** with the defect in place the outcome is `pushed` (`ahead` is true), so the test fails.
- **`PrimeCollisionIsRefusedNamingHubAndPrime`:**
  - The host is `box`, so `own` is `[box]`.
  - The name `osl46` is not in `own`, and `owners.Prime == name` returns `collision(hubA, "prime", ...)`.
  - That message contains the hub, `prime`, `"osl46"` and `lym adopt`.
  - Without the branch, the entries loop finds nothing and the set succeeds, which fails the table's refusal check.
  - The table's shared checks also cover the unchanged machine file and zero owner writes.
- **`MachineFileWriteFailureIsRefusedNamingTheFileAndTheRerun`:**
  - The first attempt put the path under a regular file, so the read failed at `readErr`. The fixer replaced it with a read-only directory that already holds the lock file.
  - The read then finds no machine file and the write fails, which gives the `write the machine file &lt;path&gt;` refusal. With that branch gone the set would succeed, so the test fails.
  - `t.Cleanup` (the chmod restore) runs before the `t.TempDir` cleanup, so the directory is removable.
  - The skip for Windows and root is correct, and the run was uid 1000, so it ran. The tier-0 run passed (39 passed, 1 skipped).
- **Replace guard rows (`internal/boardengine/ownerset_integration_test.go`):**
  - `WithSlugs`, `WithPrime` and `WithTakeover` fail the `Machine` and claim checks, then hit the guard at `ownerset.go:97`.
  - `WithClaim` carries exactly one slug, so it passes the claim check and then hits the guard.
  - Each error contains `replace set`.
  - Without the guard, `WithSlugs` and `WithPrime` would write and push, and `WithClaim` and `WithTakeover` would take the writing path. That fails either the error assertion or the local and origin head equals-seed checks.
  - The tier-1 run passed.
- **`OwnerRequest` comment:** only the doc text changed, with no behaviour change.

Further tool calls: none. The context already held every file and test run.
