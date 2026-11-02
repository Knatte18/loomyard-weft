```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the `decodeTier`, `renderTierCommand` and `tierFingerprint` changes.

**Per-function list (each case traced from the code in context)**

- `decodeTier`, command without `{test_timeout}` and no `test_timeout_min`: `holdsTestTimeout` is false, no refusal, and `tier.TestTimeoutMin` stays 0. Correct.
- `decodeTier`, command with the placeholder and `test_timeout_min` unset: `TestTimeoutMin` becomes `DefaultTestTimeoutMin` (20). Correct.
- `decodeTier`, command with the placeholder and `test_timeout_min: 25`: `TestTimeoutMin` is 25. Correct.
- `decodeTier`, `test_timeout_min: 0` or a negative value with the placeholder: refused with `0; want at least 1`. Correct.
- `decodeTier`, `test_timeout_min` set on a command without the placeholder, including `0`: refused with the "holds no {test_timeout}" message first. The placeholder refusal runs before the below-1 refusal, so this is deterministic and matches the test row. Correct.
- `decodeTier`, `{test_timeout_x}` or other near-miss text: `strings.Contains` on the exact `{test_timeout}` token does not match. Correct.
- `decodeTier`, failure paths: each refusal returns `Tier{}` before any state is written, so nothing is left half-set. Correct.
- `renderTierCommand`, command `go test -timeout {test_timeout} {targets}` with `TestTimeoutMin` 25: renders `-timeout 25m`. Correct.
- `renderTierCommand`, a target or arg containing the literal `{test_timeout}`: `strings.NewReplacer` makes one pass and does not re-expand substituted text. Correct.
- `renderTierCommand`, a command without the placeholder: the replacer is a no-op for that pair. Correct.
- `renderTierCommand`, a `Tier` with the placeholder but `TestTimeoutMin == 0`: renders `0m`, which `go test` reads as no timeout. This can't come from `decodeTier`, so it is reachable only by hand-built tiers. Not a defect on the load path.
- `renderTierEnv`: env values still fill only `{parallel}`, not `{test_timeout}`. This matches the docs, which scope the placeholder to the command. Correct.
- `tierFingerprint`, `TestTimeoutMin` varied (20 vs 45) with the same command text: the hash is unchanged, because the placeholder stays unrendered in `Command` and the field is not in the declaration struct. A tier that sets none of the new key keeps its old fingerprint, so existing green entries still carry. Correct and consistent with `timeout_min`. A tightened timeout will not invalidate an earlier pass; the commit message states this on purpose.

**Findings:** none.

**Tool calls:** none. The diff and `green.go` in context were enough.
