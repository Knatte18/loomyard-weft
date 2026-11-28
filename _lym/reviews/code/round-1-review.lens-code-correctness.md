```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each branch the diff adds or changes in `TestGateRedFirst_Dotnet` with concrete inputs.

**Branch list**

1. **Shared set-up before the loop** (`files`, `gateYAML`, the deletes of `_lym/config/gate.yaml` and `.gitignore`, `baseCircle`, `baseTests`). Case: two parallel rows reading `files` at once. `files` is written only before `t.Run`, and `plankit.WriteTree` only ranges over it, so the rows make concurrent reads only. `go.mod` is `go 1.26`, so each iteration binds its own `tt`. Correct.
2. **Per-row hub** (`hubforge.CopyHub`, the gate config write to `configengine.ConfigFile(h.BoardDir(), "gate")`, `PrimeWorktree`, the commit, `RevParse`). Case: two rows in parallel with `slots: 1`. The slot pool sits under each hub's own `BoardDir` (`internal/gateslot/pool.go:118`), so the rows do not wait on each other's slot. `lymtemp.MkdirTemp("red-first-")` gives each call its own checkout directory. `ExecuteIn` passes the cwd through the context and does not chdir (`internal/clihelp/exec.go:217-225`). Each row builds its own `Command()`. No shared mutable state. Correct.
3. **Building `Circle.cs`**. Case: row 1 with an empty `extraMember`. `3.14 * Radius` becomes `System.Math.PI * Radius`, and the closing `}\n` is trimmed and put back, the same text the old `writeCircle("")` wrote. Case: row 2 with the `Perimeter` member, which lands inside the class the same way the old helper put it. Correct.
4. **Building `CircleTests.cs`**. Case: zero methods gives the base file unchanged. One method gives the same bytes as the old `writeTest`. Two methods give two `[Test]` blocks, each preceded by a blank line, and the class's closing brace stays last. Correct.
5. **Writing over committed files with `plankit.WriteTree`**. `os.WriteFile` truncates an existing file (`internal/testkit/plankit/plankit.go:98`), so the working tree differs from `base` in exactly those two files. Correct.
6. **Envelope parse**. Case: empty output. `Split("", "\n")` returns `[""]`, so `json.Valid` is false and the row fails at once. Case: tier output on stderr. `ExecuteIn` merges stderr into `out`, and `output.Ok` or `output.Err` is the last thing written, so the last line is the envelope. Correct.
7. **Exit check `code != 0`**. Both rows expect a red test, so exit 0 is the right expectation (`internal/gatecli/redfirst.go:226-229`). Correct.
8. **`wantContains` loop on row 1, `OnlyTheTestThatFailsOnTheBaseIsNamedRed`**. Case: the dotnet run marks both tests red. `red_tests` becomes `["…AreaIsPi","…RadiusIsOne"]`, which does not contain `…AreaIsPi"]`, so the row fails as it should. Case: `RadiusIsOne` is never selected or never runs, for example because the selection drops the second added method. Then `red` is still `[AreaIsPi]` (`internal/gatecli/redfirstdotnet.go:321-327`), the exit is 0 and the needle matches, so the row passes. The row pins that `RadiusIsOne` is not named red. It does not pin that it was selected and run and passed on the base. The envelope's `tests` key (`internal/gatecli/redfirst.go:227`) would show the selection, and nothing asserts it. **Gap** (finding 1).
9. **`wantContains` loop on row 2, `ATestThatReadsAMemberTheChangeAddsIsRedSinceItDoesNotBuildOnTheBase`**. It checks the same needles as the old combined condition, now as separate errors with separate messages. Case: no `build_errors` key, and that row fails. Correct, and no looser than before.

**Findings**

1. **LOW · design** — `internal/gatecli/redfirstdotnet_integration_test.go`, row `OnlyTheTestThatFailsOnTheBaseIsNamedRed`. The row's `wantContains` holds only the `red_tests` needle. It still passes when `RadiusIsOne` is never selected or never run on the base, so it does not prove that a real passing test was judged not red. Evidence: `internal/gatecli/redfirstdotnet.go:108` builds `labels` from the selectable tests, `redfirstdotnet.go:321-327` builds `red` from failed names only, and `internal/gatecli/redfirst.go:227` puts `"tests": labels` in the envelope with no row asserting it. The fix is to add a `"tests":["Fixture.Tests.CircleTests.AreaIsPi","Fixture.Tests.CircleTests.RadiusIsOne"]` needle. Another lens or the handler may also raise this.

No other correctness defect turned up in the changed code.

**Tool calls**

- `grep "^go " go.mod` and the `plankit.WriteTree` definition: to confirm per-iteration loop variables and overwrite semantics, which the context lacked.
- `internal/clihelp/exec.go:200-239`: to confirm `ExecuteIn` merges stderr into `out` and never changes the process cwd, which the context lacked.
