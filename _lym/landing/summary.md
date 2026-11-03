# Config --set replaces a whole open map from a flow mapping

`lym config drift --set 'capture={command: "...", dir: "...", timeout_min: 30}'` was refused with "unknown config key(s): capture, compare", although the key description in `--resolved` prescribes that form.
Only the per-key form (`capture.command`, `capture.dir`, `capture.timeout_min`) worked.
A declared open map that holds a mapping now takes a YAML flow mapping that replaces it whole, so the prescribed form is accepted for `capture` and `compare`, and for every other module's open map of the same kind.

Closes #19

## Behavior

- `--set <map>={...}` replaces the map whole, `{}` included, so it also empties one.
  Missing parent mappings on the way are created, and the line comment the template carries on the map is kept.
- A value that is not one flow mapping is refused naming the key and the per-key form `<map>.<key>`.
- Under a declared shape (drift's `capture` and `compare`, gate's `tiers`), a key inside the value that the shape lacks is refused naming it, as in `capture holds no key timeoutt`.
  So is a mapping or list where the shape takes a string.
  Nothing is written on a refusal.
- Scalar tags follow the shape, so `timeout_min: 30` stays a number and a string field stays a string.

## Design

- `yamlengine.SetValues` and `SetSparse` share one helper, `setWholeOpenMap`, so the default layer and the sparse local layer behave the same.
- The open map itself is now a known key: `knownKeys` lists `P` beside `P.<name>`, and `unknownKeys` accepts `P`.
  The duplicate this creates for a key that is both a scalar leaf and an open map (such as `rounds` in the test template) is removed with `slices.Compact`.
- In `SetSparse` the scalar-or-list branch and the whole-map branch run before the shaped-entry branch, and the parent-mapping lookup moved into `parentMappingOf`.
  A key declared both ways keeps its previous scalar-or-list handling.
- The line-comment move in `setWholeOpenMap` exists because a value's line comment is dropped when a block mapping marshals.
- Docs updated in `yamlengine`, `configengine`, `configcli` and `driftengine.ConfigSpec`.

## Tests

New cases cover `yamlengine` set and sparse set (whole replace, empty, shaped tags, shape refusals, non-mapping refusal, dotted parent creation, comment kept, known-key listing without duplicates) and `driftengine.TestConfigSpec_SetWholeSteps`, which sets both steps whole and loads the file.

## To check by hand

- Not run: `lym config drift --set 'capture={command: "...", dir: "...", timeout_min: 30}'` against a real project, including `--local` refusal of the executable keys.
- Check that no other module declares an open map whose callers relied on `P` being refused as unknown.
