# Card 1 — Pin board.json's encoding before state starts keeping keys

**Edit:**
- `internal/boardengine_test#TestLoad`

**Intent:**
Pin the exact bytes `Store.Save` writes for `board.json`, so the key-keeping merge card 3 adds to `internal/state` cannot change them: `storeFile` and `Task` implement `json.Marshaler`, and D2 makes such a type opaque, written whole.
This card lands first, on the tree before any `state` change, so the pinned bytes are the encoding of `78e31aa06`'s `storeFile` and `Task` Marshalers as merged here.

Add one field to `TestLoad`'s table, `saveExact string`: when set, the saved file must equal it byte for byte.
Set it on two existing rows, adding no row: "the old tier-and-type shape converts on Load and Save writes kind and labels", whose retired `tier` and `type` the Marshaler consumes, and "set-status keeps kept keys after the known fields in key order", whose unknown top-level and entry keys the Marshalers write after the known fields.
Each `saveExact` is the full indented file `Save` writes on this tree: run the row once with a placeholder to read the bytes, paste them in, and check by eye that they match the row's existing `saveContains` and `saveAbsent`.
Without opacity, D2's merge would carry a retired key the Marshaler consumed, or re-order the kept keys, and the exact comparison fails.

The table's existing doc line on `save`, `saveContains` and `saveAbsent` gains the `saveExact` field.

**ImpactSummary:** one test table in `internal/boardengine` gains a field set on two rows; no production code changes.

**Commit:** 1: pin board.json's Save encoding byte for byte
