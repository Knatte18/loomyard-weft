no findings

- The constant is still a single-line string with no quote, backslash or line break. `settings.go:462` checks it against the forbidden characters at init, and the added clause adds none.
- All uses of `batchingSentence` (`settings.go:450`, `settings_test.go:803`, `prepare_test.go:398` and `:520`) build their expectation from the constant, so the changed text breaks none of them.
- The gate test for `./internal/shuttleengine/claudeengine` passed in the parent's session.
- The new doc comment is two `//` lines directly above the constant, so it parses as one godoc comment.

Tool calls (1):
1. `grep -rn "batchingSentence"` over the repo's Go files. My context showed only the constant and `appendPromptText`, not the other callers or tests.
