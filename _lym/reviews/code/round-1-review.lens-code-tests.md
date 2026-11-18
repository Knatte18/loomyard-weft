```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests through the code-tests lens.

**Per behaviour, the test that pins it**

1. **Every plain steer opens with `wholeCommandRefusal`** (`internal/shellguard/rows.go:389`).
   - `TestRows_SteersOpenWithWholeCommandRefusal`, through its `claude` pass: `SteerFor("claude")` returns the plain steer, and `TestRows_EngineTexts` at `rows_test.go:136` pins that.
   - The claudeengine golden test (`settings_test.go:1125` against `settings-golden.json`) also covers it: the golden gains the clause on every plain steer and nothing else.
2. **Every Codex engine steer opens with the clause** (`rows.go:390-394`).
   - `TestRows_SteersOpenWithWholeCommandRefusal` covers it through its `codex` pass.
   - For python alone, `TestCheckTexts` covers it as well (`rows_test.go:54`, `wholeCommandRefusal+pythonRefusal`).
3. **The refuse-raw row's steer carries the clause.**
   `TestRows_SteersOpenWithWholeCommandRefusal` builds `Rows` with `[]string{"go test"}`, so the refuse-raw row is included. The golden covers it too.
4. **The refused part follows the clause and is not empty.**
   `TestRows_SteersOpenWithWholeCommandRefusal` checks this with `refusedPart == ""`.
5. **Notices stay free of the clause.**
   - Plain notices: the golden (its notice entries are unchanged after the strip) and `TestRows_SteersOpenWithWholeCommandRefusal` on `claude`.
   - Codex notices: only `TestRows_SteersOpenWithWholeCommandRefusal` on `codex` checks them. Those are the shell-edit and headless-claude Codex notices, which are built from their unprefixed Codex steers.
6. **The raw-go notice is its steer minus the clause, plus a full stop.**
   `TestRows_RawGoSteerNamesOnlyUnrefusedCompileChecks` pins it (`rows_test.go:173`).
7. **The clause carries no forbidden character.**
   The package `init` runs `checkTexts(Rows(...))` over the prefixed rows (`rows.go` `init`), so every test in the package fails at load if it breaks.
8. **An engine text with an empty `Steer` stays unprefixed** (the `if text.Steer != ""` branch at `rows.go:391`).
   - No test pins this. Every shipped `EngineTexts` entry carries a `Steer` (`rows.go` python, shell-edit and headless-claude rows), so no row reaches the branch.
   - `Rows` takes no row data, so the public surface cannot reach it either. Whether the branch should exist belongs to the correctness lens, so I raise no test finding.

**Findings**

- **Severity:** LOW. **Class:** consistency.
  - **Location:** `internal/shellguard/rows_test.go:183` (`TestRows_SteersOpenWithWholeCommandRefusal`).
  - **Summary:** The redundancy report lists this test as contained by `TestCheckTexts` and `TestRows_EngineTexts`, but neither asserts what it pins.
    - `TestCheckTexts` checks only the python Codex steer's prefix (`rows_test.go:54`).
    - `TestRows_EngineTexts` checks tool names, the edit rule and the Claude fallback (`rows_test.go:124-141`), never the clause.
    - What only this test pins: the clause opens the shell-edit and headless-claude Codex steers, and it is absent from the Codex notices.
  - **Fix:** Keep the test and mark it `//testtiming:keep` directly above its `func Test…` line. The reason should say it pins that every steer on both engines opens with the whole-command clause and that no notice carries it.
  - **Evidence:** the redundancy report in the prompt; `rows_test.go:118-146`, `:51-56`, `:183-199`.

Tool calls:
1. I grepped `TestRows_EngineTexts`, the golden's notice entries, the golden test's site and the Codex tests that read steers. My context lacked the covering test's assertions, which I needed to judge the redundancy candidate.
