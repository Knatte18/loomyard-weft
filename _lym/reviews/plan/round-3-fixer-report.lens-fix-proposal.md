```yaml
lens: fix-proposal
outcome: reported
```

I challenged the five proposals the fixer carried out, F1 to F5.

**LOW**: card 6's new engine-name constants paragraph (F5) leaves card 8's `shuttlecli` test without a source for the name.
- **Changed text:** "The package declares the reference engine's name, `claude`, and the compared engine's name, `codex`, once each as named constants, and `Compare`, the manifest keys of `PrepareEach`, card 8's `Report` and every test of the package use them."
- **Summary:** Card 8's `TestManifestCmd` lives in `internal/shuttlecli`, not in this package, and the card never says whether the constants are exported, so that test is left adding a fresh `codex` literal.
- **Evidence:** Card 8's `TestManifestCmd` row says the stub's `ManifestReport` row "prints `ok` naming `codex` and the requirement". The stub's `Unmet` map key and that expectation need the engine name, but "every test of the package" reaches only the `shuttleparity` tests. `PATTERN-code-conventions` says a change never adds another literal for a name that can go stale. The fix should state that the constants are exported and that `shuttlecli`'s test uses them.

Proposals I built cases against that held:
- **F3:** Clearing `Model`, `Effort` and `Version` is safe:
  - Claude's `resolveModelID` and `validateEffort` and Codex's `validateChoice` and `validateEffort` all accept empty values.
  - Cards 1 to 4 derive no manifest item from model or effort, so clearing them hides no item that should be compared.
- **F2:**
  - Allowlist keys take the form `&lt;directory&gt;.&lt;Type&gt;.&lt;Method&gt;`, which matches the `unreachableFunction.key` doc, so `internal/burlerengine.Engine.SeatTables` is well formed.
  - The guard uses RTA, which follows the function values in `seatTables`' `Build` closures once card 8 makes `Report` reachable. So all of card 6's entries do go stale at card 8, and deleting them there is correct.
  - `RequireNoStale` also confirms that each interim entry is live while its function is unreachable.
- **F1:** `TestMain` mirrors `seatengine`'s. The package's Codex `Prepare` path checks the checkout, so the git hermeticity applies to it too.
- **F4:** The new `build` signature matches the `constructor` change, and `New` and `NewWithSeams` keep theirs.

Tool calls:
1. Grepped `codexengine` `validateChoice` and `claudeengine` `resolveModelID`/`validateEffort`: needed to know how empty model fields behave, which my context lacked.
2. Grepped the allowlist for method keys and the stale-entry check: needed the key format, which my context lacked.
3. Read lines 60–110 of `testonlycode_linux_test.go`: needed the key doc and the `RequireNoStale` call, which my context lacked.
4. Grepped the engines for `ManifestItem{` and Codex `validateEffort`: needed to know whether any manifest item depends on effort or model, which my context lacked.
5. Grepped cards 1 to 4 for effort and model: needed to know whether any planned manifest item depends on them, which my context lacked.
6. Grepped the guard's call-graph imports: needed the algorithm (RTA) to judge whether closure callees count as reached, which my context lacked.
