```yaml
lens: code-correctness
outcome: reported
```

Lens code-correctness: I traced each function and branch the diff adds or changes, running one concrete case through each.

**Per function or branch: the case run, and the result**
- **`internal/configcli/configcli.go:516` `buildConfigLong`**: `configengine.KeyName("shuttle", "deny_python")` returns `"shuttle.deny_python"` (`internal/configengine/config.go:106-107`). The help line reads `shuttle.deny_python=false lets Python through…`, and the `--set deny_python=false` line below it keeps the in-module path. Correct.
- **`buildKeyNameIndex`, a template that is empty or holds only comments**: `document.Content` is empty, so the module is skipped. Correct.
- **`buildKeyNameIndex`, a seed-only module (`models`, `profiles`)**: skipped. Correct.
- **`buildKeyNameIndex`, a bare form shared across modules or across paths of one module**: loom's `max_bounces` appears under discussion, review, code and split, and hardener has it too. Every full name is appended. Correct.
- **`templatePaths`, a key whose value is a flow `{}` (`gate.build`, `drift.capture`)**: an empty mapping returns no children, and the key's own path is indexed. Correct.
- **`templatePaths`, a scalar node**: returns nil. Correct.
- **`templatePaths`, an open or per-name map (`gate.tiers`, `gate.gates`)**: indexed, and recursion stops. Correct.
- **`bareKeyNames("loom.code.max_bounces")`**: the regex takes the whole dotted chain, which is not in the index, so the name passes. Correct.
- **`bareKeyNames("gate.yaml's refuse_raw")`**: `refuse_raw` is its own token, preceded by `s `, so it is flagged. Correct.
- **`bareKeyNames(".timeout_min")`**: before is `.`, so it is skipped. As designed.
- **`bareKeyNames("the `scoper` key")`**: flagged. For plain `the scoper key`: not flagged. As documented.
- **`bareKeyNames("--set review.max_bounces=3")`**: skipped. Correct.
- **`bareKeyNames("max_bounces: 3")`**: skipped as a document field. For a prose message such as `"max_bounces: must be at least 0"`: also skipped. This is the miss in finding F1.
- **`bareKeyNames(`--unset "max_bounces"`)`**: `--unset "` is missing from `qualifiedPrefixes`, while `--set "` is present, so the name would be flagged. This is a false positive, not a miss (F3).
- **`bareKeyNames("beam.max_bounces")`, wrong module**: the whole token is not in the index, so it passes. The guard cannot catch a wrong module. That is outside what it claims to do.
- **`bareKeyNames("foo.max_bounces")`, a non-module word joined by a dot**: the whole token is not in the index, so it passes. This is a small miss, and no production instance was seen.
- **`isKeyNameCall` on `configengine.KeyName(...)`, a local `keyName(...)` or `x.fooKeyName(...)`**: all return true. Any argument literal is then skipped, including literals nested deeper in the arguments. Acceptable.
- **`stringLiterals`, import declarations**: skipped. Correct.
- **`stringLiterals`, `*ast.Field`**: covers struct tags and parameters. A function body is not a `Field` child, so no body literal is lost. Correct.
- **Test loop, a raw-string literal**: `strconv.Unquote` handles backtick literals. Correct.
- **Test loop, a literal that is one key path whole (`"max_bounces"`)**: skipped as data. A message split across concatenated literals with the key name in its own literal, such as `"invalid " + "code.max_bounces"`, therefore passes. This is a small miss.
- **Test loop, a `_test.go` file**: `scankit.Walk` with a zero `Options.Filter` is `Prod` and excludes it (`internal/testkit/scankit/scankit.go:26-27,214-218`). So the allowlist prose in the guard file itself is not scanned. Correct.
- **Test loop, an allowlist match**: the match keys on `<file>: <name>`, so every literal in that file naming that name passes (F2).
- **`RequireFloor` and `RequireNoStale`**: a stale entry fails the test. Correct.

**Findings**
- **F1 — LOW, design**
  - Location: `internal/configreg/keynames_enforcement_test.go`, the colon skip in `bareKeyNames`.
  - Summary: a bare key followed by a colon is skipped wherever it stands, so the common Go error form `"<key>: <problem>"` passes the guard even though it names a key bare in a message.
  - Evidence: the `strings.HasPrefix(strings.TrimLeft(after, …), ":")` branch. A search of production messages found no current instance; the one hit, `internal/driftengine/config.go:397` `timeout_min: N`, is a document example.
- **F2 — LOW, design**
  - Location: `internal/configreg/keynames_enforcement_test.go`, `bareKeyNameAllowlist` and its `allowlist.Allowed(f.Rel + ": " + bare.name)` check.
  - Summary: entries are keyed by file and name, not by literal. A new message that names `timeout_min` bare in `internal/gatecli/tierrun.go` or `internal/gateslot/runlog.go`, or `driver_kind` in `internal/battencli/keynames.go`, would pass unseen.
  - Evidence: the allowlist entries and the `Allowed` call in the test loop.
- **F3 — NIT, design**
  - Location: `internal/configreg/keynames_enforcement_test.go`, `qualifiedPrefixes`.
  - Summary: `--unset "` is missing beside `--set "`, so a double-quoted `--unset` path argument would be reported as bare. That is a false failure, not a miss.
  - Evidence: the `qualifiedPrefixes` slice.

**Tool calls**
1. Read `scankit`'s test-file filter and `configengine.KeyName`, to confirm that test files are excluded and what the help text renders. The manifest held neither definition.
2. Searched production literals for a bare key before a colon in messages, to check whether F1 has a live instance. The manifest held no such search.
3. Read `scankit`'s `FileFilter` constants and the filter switch, to confirm the zero value is `Prod`. The manifest lacked them.
