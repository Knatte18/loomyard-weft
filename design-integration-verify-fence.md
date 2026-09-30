On the first real lyx loom run (task `burler-exclude-warn`, PR #280), Webster's integration fork reported that the verify command in its rendered prompt `integration.md` was mangled by nested code fences, so it ran the plan's own `## verify:` command from `00-overview.md` instead.

The fork recovered by itself, but the integration stage's prompt carries the one command that decides whether the whole plan passes: a fork that trusts the rendered text runs something other than the plan's verify, and one that cannot recover it fails the stage.

Expected: the verify command renders into `integration.md` verbatim and unambiguous, whatever fences the plan's command or its surrounding prompt text use (e.g. a fence longer than any backtick run inside the command, or the command outside a fenced block).

## Second occurrence

Second occurrence, and it is wider than `integration.md`'s rendering. On the `stuck-reason-in-status` run (llm driver), the plan put its `## verify:` command inside a ```bash fence, and Webster ran the fence lines as part of the command:

```
websterengine: verify command exited verifyCmd="```bash && go build ./... && go vet ./... && go test ./... && go test -tags integration ./... && ```" exitCode=2
```

The plan-level verify is joined line by line with `&&` fence markers included, so the shell fails on syntax (exit 2) at every commit. That also makes the integration stage's SHA-bisect useless: every candidate fails the same way, so it can never localize the commit that actually broke (here a real test failure was masked this way).

Fix belongs where `plan.Verify` is extracted (planparser), not only in the prompt render: strip a fenced block's markers, or reject a fenced verify section at the plan gate so the planner rewrites it.

## Where the fix goes

`internal/planparser/sections.go`, `joinVerifyCommands`: skip markdown code-fence lines (a trimmed line starting with ``````` or `~~~`, with or without a language name) when chaining the section's lines, so a fenced `## verify:` yields the same command as a bare one.
Add fenced cases (with language, bare, tilde, fence-only) to `TestParsePlan_VerifySection_ChainsEveryLine` in `sections_test.go`, and state the rule in `contracts/specs/loom-plan-spec.md` next to "the parser chains every non-blank line".

## Constraint for this run

The deployed lyx still has this bug, so this plan's own `## verify:` section MUST be one plain command line with no code fences, or this run blocks on it exactly like `stuck-reason-in-status` did.

Source: https://github.com/Knatte18/loomyard/issues/285 (close it in the landing commit).