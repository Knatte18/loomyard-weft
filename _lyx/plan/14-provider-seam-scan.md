# Card 14 — provider-seam-scan

**Edit:**
- `internal/shuttleengine#TestProviderSeamImportRule`
- `PATTERN.md`
- `pattern/PATTERN-shuttle-provider-seam.md`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/engine.go`

**Intent:** Amend `PATTERN-shuttle-provider-seam` to the registry rule and widen its scan to match (D13), now that no production file outside `internal/shuttleengines` and the engine's own directory imports a concrete engine.

`TestProviderSeamImportRule` replaces its `shuttleengine`-only `claudeengine` denylist with a module-wide scan of production `.go` files (`scankit.Walk` over the module root with the default production filter).
A file violates the rule when it imports a package under `github.com/Knatte18/loomyard/internal/shuttleengine/` while it lies outside that engine's own directory tree and outside `internal/shuttleengines`.
Test files are out of scope and keep importing `claudeengine` directly.
The scan body becomes a helper over a base directory, and the test is table-driven over it: a temp tree whose `internal/x/x.go` imports `internal/shuttleengine/claudeengine` reports that file; a temp tree whose import sits in `internal/shuttleengine/claudeengine/a.go` or `internal/shuttleengines/b.go` reports none; and the real module reports none, with `scankit.RequireFloor` on the files scanned.
Its `//testtiming:keep` reason becomes: scans every production file's imports for a concrete engine package outside its own directory and the registry, which no other test reads.
The per-package allowlist scans in `cliwire` and `orchengine` stay as they are, `cliwire`'s already shrunk by card 5.

`PATTERN.md`'s entry becomes, under the 320-rune cap:

```markdown
- `PATTERN-shuttle-provider-seam` — Referencing a provider: its specifics live only under `internal/shuttleengine/<engine>`, and only `internal/shuttleengines` constructs or imports a concrete engine. (test) — [background](pattern/PATTERN-shuttle-provider-seam.md)
```

`pattern/PATTERN-shuttle-provider-seam.md` states the same rule in its opening line and adds two bullets:
only `internal/shuttleengines` imports or constructs a concrete engine, the scan in `internal/shuttleengine`'s `seam_enforcement_test.go` covering production files;
and an engine's name, a registry key such as `shuttle.yaml`'s `engine: claude` or a `modelspec` engine name, is not a provider specific and may appear outside its directory.
Its existing bullets stay, the first reworded from `claudeengine` to "a concrete engine".

`internal/shuttleengine/doc.go`'s sentence saying the provider-seam rule is "enforced by seam_enforcement_test.go in a later batch" becomes the current module-wide rule, and `engine.go`'s file header and `Engine`'s doc comment name the registry rule instead of "never imports a concrete engine" alone.

**ImpactSummary:** One test widened from one package to the module, one PATTERN entry with its background reworded, and shuttleengine's seam comments updated; no production code changes.
