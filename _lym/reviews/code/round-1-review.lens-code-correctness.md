```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over a diff that changes only `PATTERN.md` and two `pattern/` background files.

**List (function/branch → case → result)**

The diff adds no Go code, no function, no branch and no error path, so there is nothing to run step by step. The only executable-like surface is the mechanical gate `lym loom lint-pattern`, which the summary says passes. That gate is outside this lens, and I did not re-run it.

- Go functions or branches changed: none. No case to run.
- `PATTERN.md` entry format: each new line follows `` `PATTERN-x` — rule. — [background](pattern/PATTERN-x.md) `` or omits the link. The link targets `pattern/PATTERN-fabric-isolation.md` and `pattern/PATTERN-no-hard-coded-choice.md` both exist in the diff. The three entries with no background file carry no dangling link. Correct.
- Factual claims in the background files, checked against the repository directory listing already read:
  - `internal/fabricengine` and `internal/fabriccli` exist. Correct.
  - `shuttleengine` and `shuttleengines` exist, and `PATTERN-shuttle-provider-seam` is an existing entry. The "shuttle's provider seam" claim is correct.
  - The "debt" statements describe existing imports and literals. The summary says fabric imports packages of the board, the IDE, heddle, reed and webster, and the plan parser reads an absent language as `go`. I did not verify those claims against the source, because that belongs to the affected-set lens and the handler. Not judged.

No findings under this lens.

**Tool calls:** none made. My context already held everything this lens needed.
