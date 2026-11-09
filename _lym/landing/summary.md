# config profiles: --set takes a dotted key to a leaf value

`lym config profiles --set` took only a whole profile, keyed by its name alone.
`--set docs.darn.loom.code_max_bounces=0` was refused with a message that named a value shape fitting a recipe block, not a leaf key.
Following that message with `docs.darn={...}` failed too, so the only working form restated every block of the profile.

A dotted key now sets the one value it names inside a profile and keeps the rest of the profile, comments included.
This is the behaviour `lym config gate --set tiers.tier0.env.X=...` already has in other modules.

Closes #94

## What changes

- `yamlengine.SetEntries` accepts a dotted key.
  It creates every missing mapping on the way and keeps the comments of the key it sets.
- A value starting with `{` or `[` is written as a flow mapping or list.
  An empty value is an empty string.
  Any other value is a scalar under the type its text spells, so `0` is a number, `true` a boolean and `sonnet[medium]` a string.
- An undotted key still takes a JSON object and replaces the entry whole.
  Its refusal now says the key names a whole entry and points at the dotted form, such as `docs.<key>.<key>=1`.
- A key with an empty segment, and a dotted key through a value that is neither a mapping nor null, are refused with an error naming the path.
- The docs for `configcli`, `configengine`, `runprofile`, `yamlengine` and `docs/overview.md` describe the dotted form.

## Design decisions

- A dot in a key always separates levels, so a profile name holding a dot cannot be set by `--set`.
  Setting such a name needs the editor, `lym config profiles`.
- The dotted path is implemented in `SetEntries`, so every spec with `JSONEntries` gets it, not only `profiles`.
- A dotted write that leaves the profile in a wrong shape, such as an unknown key, is refused by the existing profile check, and the file is restored.

## To check by hand

Nothing was left unrun.
