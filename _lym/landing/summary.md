# Name config keys in full in Go comments and guard string literals

Since the config keys were nested by area, many Go comments still named a template key without its module,
against `PATTERN-config-key-names`.
Comments and package docs under `cmd/` and `internal/`, in production and `_test.go` files, now write `<module>.<path>`.

A new test guards against the same drift in messages and help texts, where no tool detected it.
`TestKeyNames_StringLiteralsNameEveryConfigKeyWithItsModule`,
in `internal/configreg/keynames_enforcement_test.go`,
fails on bare template key names in production string literals under `cmd/` and `internal/`.
The guard is partial: most mentions of a one-word key such as `engine` or `scoper` pass,
as does a name followed by a colon and one word.
`PATTERN-config-key-names` is tagged `(test)` accordingly.

One user-visible text changes: the `lym config` long help now writes `shuttle.deny_python=false`
where it wrote `deny_python=false`.
The other edits to existing Go files are in comments.

Closes #265

## Design decisions

- The guard reads each registered module's template, seed-only modules left out,
  and treats a key's dotted path, or any trailing part of it, as a bare form.
- A one-word key counts only when written in backticks before the word `key`, since words such as `engine` are also prose.
  Other bare one-word keys pass.
- The guard skips a literal that is one key name and nothing else, struct tags, import paths,
  arguments of the key-name rendering helpers, a name right after `.` or `%`,
  `--set` and `--unset` path arguments, and fields of document examples.
- A name reads as a document field when its colon is followed by nothing at all or by exactly one word.
  The value ends at the line end, or inside an open brace at a comma or closing brace.
  A one-word problem such as `max_bounces: required` therefore passes,
  while `max_bounces: ` with text joined on later is reported.
- Literals that use a key's name for something else, such as a recipe row field or a log-line format,
  are allowlisted by file, name and literal text, and a stale entry fails the test.
- The guard checks string literals only, so nothing keeps comments in full form mechanically.
- Some comment names stay bare on purpose: recipe row and seed fields, retired board entry fields,
  drift step fields, prompt markers, state and envelope fields, YAML examples,
  and generic mentions where no single module fits.

## To check by hand

- The detector's own test, `TestKeyNames_DetectorReportsOnlyBareNames`
  in `internal/configreg/keynames_enforcement_test.go`, leaves some skips without a case.
  Among them are a literal that is one key name whole, the `.` and `%` prefixes, and `--set=`.
- `_test.go` comments under `cmd/` and `internal/` were swept by search, so check them for bare key names left over.

## Merge danger

**Door:** two-way
**Blast radius:** tests
