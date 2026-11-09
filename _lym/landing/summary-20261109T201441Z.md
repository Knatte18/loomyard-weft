# lym config profiles --set reaches one value of a profile by a dotted key

`lym config profiles --set` took only a whole profile, keyed by its name alone, with a JSON object.
A dotted key such as `docs.darn.loom.code_max_bounces=0` was read as a new top-level entry literally named `docs.darn.loom.code_max_bounces`.
Its value was refused because it was not a JSON object, and the refusal suggested `{"module": {"key": 1}}`, a shape that fits a recipe block, not a profile.
A JSON object under a dotted key was accepted but stored one level too deep, so the profile load then refused it.

## What changes

`yamlengine.SetEntries`, the write path of every `JSONEntries` config module, now reads a dotted key as a path below a top-level entry.
It sets that one value, keeps the rest of the entry, and creates any mapping missing on the way.
So `lym config profiles --layer default --set docs.darn.loom.code_max_bounces=0 --set docs.darn.loom.code_hard_max_bounces=1` changes those two values and nothing else.

The value of a dotted key follows the rule that the shaped open maps of other modules already use:

- a value starting with `{` or `[` is a flow mapping or list, written at the key's own depth;
- an empty value is an empty string, so `code_reviewer_fan=` writes `""`, not a null;
- any other value is a scalar of the type its text resolves to, so `0` is a number and `sonnet[medium]` a string.

A key without a dot keeps its old meaning, a whole entry set from a JSON object.
Its refusal now says the key names a whole entry and gives the dotted form, `<name>.<key>.<key>=1`, as the way to set one value.
A key with an empty segment is refused.
So is a dotted key that passes through a value that is not a map, and that refusal names the path and says to set it whole or rewrite it as a map.

The profile load check that `lym config` runs after every write is unchanged.
A dotted key that leaves a profile in the wrong shape is therefore still refused, and the file is put back.

## Design decisions

- The change lives in `yamlengine`, not `runprofile`.
  `SetEntries` knows nothing of recipes or modules, so the refusal names the dotted form generically, not as `<recipe>.<module>.<key>`.
- No template key check is added on the dotted path.
  A profile's keys are checked by the load that follows the write, as before.
- The header comment of `internal/runprofile/template.yaml` still names only the whole-profile form.
  It is seeded into operators' `profiles.yaml` files, and changing it would change the seeded file for a comment.
  The package doc, the `configcli` doc and `docs/overview.md` name the dotted form.

## Tests

`TestSetEntries` in `internal/yamlengine/set_test.go` gains rows that cover:

- a dotted key setting one value inside a flow-style entry and keeping the rest;
- a dotted key creating the mappings it lacks and writing a number as a number;
- a string value that holds brackets, and an empty value;
- a braced value written at the key's own depth;
- a dotted key through a scalar being refused;
- an empty segment being refused.

The existing refusal row now also checks that the error names the dotted form.
