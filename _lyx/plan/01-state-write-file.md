# Card 1 — state-write-file

**Create:**
- `internal/state#WriteFile`
- `internal/state#TestWriteFile`

**Intent:** Add `WriteFile` to `internal/state`: it writes raw bytes to `path` atomically under the write lock at `lockPath`, creating the parent directory, as `WriteJSON` does for a JSON value but with no marshal and no unknown-key merge.
The Bouncer's settle record keeps the full task diff as a patch file beside the record (D5), and `PATTERN-tolerant-records` routes every write of a record, the patch included, through `internal/state`.

`TestWriteFile` pins: the bytes land verbatim, a second write replaces the first whole, the parent directory is created, and an unwritable path (the target is a directory) returns an error and leaves nothing beside it.

**Commit:** 1: state: WriteFile writes raw record bytes atomically under the state lock
