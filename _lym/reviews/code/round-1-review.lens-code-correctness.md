```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the scan code in `cmd/lym/interactivespawn_test.go` and `cmd/lym/fslinkonly_test.go`. Both scans match D2 and D4 on the inputs I ran, except the three findings below.

**Per-function cases**

- `isPrintFlag`: `-p`, `--print` and `--print=json` return true, and `""` and `-pp` return false. Correct.
- `namesClaudeBinary`:
  - `claude`, `/usr/bin/claude` and `C:\bin\claude.exe` return true.
  - `""` and `claude/` have an empty base and return false, with no index panic.
  - Correct.
- `stringLiteralValue`: an interpreted or raw string literal is unquoted, and a non-string `BasicLit` returns false. Correct.
- `exprNamesClaude`:
  - Ident `claudePath`, selector `claude.Binary` and call `claudeBinary()` return true.
  - `nil` returns false.
  - Paren, star, unary, index and composite wrappers fall through to false, so `exec.Command(*claudePath, "-p")` passes. This is within D2's "static shapes" bound.
- `callSpawnsHeadlessClaude`:
  - `("claude","-p","hi")`, `(nil,"/usr/bin/claude","--print")` and `(claudePath,"--print=json")` are flagged.
  - `("tmux","display-message","-p","#{pid}")` is not flagged.
  - Correct.
- `literalTokens`: `"claude --model opus -p 'hi'"` gives `[claude --model opus -p hi]`. A raw `& 'C:\bin\claude.exe' --print` gives the quote-trimmed path token. An empty literal gives an empty slice. Correct.
- `headlessClaudeLine`:
  - claude followed later by `-p` returns true.
  - `tmux display-message -p claude` (flag first) returns false.
  - `claude --version && tmux display -p` returns true. That is a false positive, but D2's "followed later" rule allows it.
  - `claude;` or `claude)` as a token does not match, which is a static-shape miss.
  - Correct per D2.
- `headlessSpawnFailures`:
  - Zero `Options.Filter` is `Prod`, so the test-file fixture yields 0.
  - Inside the claude-engine directory, `" -p"` gives tokens `[-p]` and is flagged.
  - Outside it, the same string passes.
  - `exec.Command("claude","-p")` inside that directory gets two reports, one from the call and one from the `-p` literal. This affects the failure count only.
  - The `f.FileSet()` call comes after `f.AST`, so it is valid.
  - Failures are sorted. Correct, with one allowlist defect below.
- Current tree: my grep found no `-p` or `--print` token in a non-comment, non-test line under `claudeengine`, so the directory rule should not trip on the tree.
- `linkCallFailures` (fslink scan):
  - `os.Symlink`, `os.Link` and `sys.Symlink` with `import sys "os"` are flagged.
  - A call under `internal/fslink/`, a comment mention, `os.Readlink` and `_test.go` files are not flagged.
  - The `<file>:<call>` allowlist entry is looked up only when a matching call is found, so `RequireNoStale` is accurate. The key `tools/deploy/main.go:os.Symlink` matches `tools/deploy/main.go:295`.
  - Correct apart from the dot-import gap below.
- `osImportName` (existing, `cmd/lym/baretemp_test.go:32`): no `os` import gives `""`, no alias gives `"os"`, an alias gives the alias, and `_` and `.` give `""`. It behaves as the scan assumes.

**Findings**

1. LOW, design, `cmd/lym/interactivespawn_test.go:114`: `allow.Allowed(rel)` runs for every walked file before any finding is computed, so an allowlist entry is marked used just because its file exists.
   - An entry for a file whose findings have since been fixed never reports stale.
   - That contradicts the allowlist's own comment that an entry admits findings, and D2's "reports its stale entries".
   - The list is empty today, so nothing is wrong yet.
   - The fix is to call `Allowed` only when a finding would be reported.
   - Evidence: lines 112-116 and 119-121.
2. LOW, design, `cmd/lym/fslinkonly_test.go:42-45` with `cmd/lym/baretemp_test.go:40-41`: a file that dot-imports `os` makes `osImportName` return `""`, so the scan skips it and an unqualified `Symlink(...)` call passes.
   - D4 says "through any name the file binds os to".
   - The focus asks about the dot import specifically.
   - The fixtures cover an aliased import but not a dot import.
   - Dot imports are rare, and the repo's lint may forbid them.
3. LOW, design, `cmd/lym/fslinkonly_test.go:55-67`: only a `CallExpr` is matched, so `f := os.Symlink; f(a, b)` or passing `os.Symlink` as a function value is missed. D4 says "call", so this is within the stated bound, but the background file does not name it as a bound.

**Tool calls**

1. A grep of `claudeengine` non-test Go files for `-p` or `--print` tokens, plus a read of the `scankit` filter constants. My context lacked the current tree's claude-engine strings and the `Filter` zero value.
