```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens review of plan cards 1–4.

**Per change: the affected set I found and the cards that cover it**

1. **`OwnerRequest` gains `Replace`; `SetOwners` gains a replace selection and a new commit subject** (card 1).
   - Code: `OwnerRequest`, `SetOwners`, `writeOwners`, `TestSetOwners`. Card 1 covers all four.
   - No `//lym:seam` test reflects over `OwnerRequest`. Every other caller builds it by field name, so none is affected.
   - Docs that describe the changed behaviour:
     - `internal/boardengine/doc.go:183-185` says "Without Takeover, SetOwners leaves an entry whose effective owner is another machine untouched … Only `lym adopt` passes Takeover". Card 1 lists this only under Extends, so it covers the doc partially.
     - `internal/boardengine/doc.go:174` gives the commit subject as "board: owner &lt;machine&gt; for &lt;slugs&gt;". Same partial cover.
     - The `SetOwners` doc comment (`ownerset.go:72-73`) and the `OwnerRequest.Takeover` field comment (`ownerset.go:43`). Card 1 targets both symbols but its Intent says nothing about rewriting these comments.
     - `pattern/PATTERN-owner-machine.md:27`. Card 1 covers it.
     - The `PATTERN.md` line. The declared text equals the base, so nothing changes.
2. **`machinename.HostnameName` is a new export** (card 2).
   - `internal/machinename/doc.go`. Card 2 covers it (Extends).
   - `docs/overview.md:362`, the machinename line. It stays accurate.
   - The sole-source scan only reads production files, so the new test assertion doesn't affect it.
3. **New module `machine` / `internal/machinecli`** (card 3).
   - The module list in the `newRoot` `Long` text (`cmd/lym/main.go:135`). Card 3 covers it.
   - The `seamsignature_test.go` pin lists. Card 3 covers them.
   - `registration_test.go` and `clitree_test.go` discover modules automatically and need no edit.
   - The `docs/overview.md` tree and module bullet. Card 3 covers them.
   - The Package naming deviations in `pattern/PATTERN-cli-cobra.md`. Card 3 covers them.
   - The `lym machine name` way forward already named in `machinename.go:28` and in the battencli and beamcli tests. The verb makes it real, and nothing in them needs a change.
4. **Owner sweep output and recoveries** (card 4).
   - `internal/machinecli/doc.go`. Card 4 covers it.
   - The verb is new, so nothing outside the package reads its envelope.

**Findings**

- **MEDIUM**, class `consistency`, location card 1 (`01-owner-replace-set.md`, Extends `internal/boardengine/doc.go`).
  - Summary: card 1 adds a replace-set paragraph to `boardengine/doc.go` but leaves standing text that the replace set makes false, and it lists that doc under Extends, which may only add to a ref.
  - What it missed:
    - `doc.go:183` says that without Takeover, an entry another machine owns is left untouched. A `Replace` write overwrites exactly such an entry.
    - `doc.go:185` says only `lym adopt` passes Takeover. That line stays true, but it is now the wrong boundary to describe.
    - `doc.go:174` gives the single commit subject "board: owner &lt;machine&gt; for &lt;slugs&gt;". A `Replace` write commits under "board: owner &lt;machine&gt; replacing &lt;names&gt;".
    - On the same card, the `OwnerRequest.Takeover` field comment (`ownerset.go:43`) and the `SetOwners` doc comment (`ownerset.go:72-73`) say the same thing about Takeover. The Intent does not tell the implementer to qualify them for `Replace`.
  - Fix: move `internal/boardengine/doc.go` from `**Extends:**` to `**Edit:**`. In the Intent, name the rewrite of lines 174 and 183 and of the two comments.
  - Evidence: `internal/boardengine/doc.go:174,183-185`, `internal/boardengine/ownerset.go:43,72-73`, `_lym/plan/01-owner-replace-set.md:11-12,39`.

**Tool calls**

1. Grepped for `//lym:seam` markers on `OwnerRequest`, its literal constructions, and every mention of "machine name" in code and docs. The manifest had no callers or docs for those strings.
2. Grepped for docs and specs that say only `lym adopt` takes over or that describe Takeover, and for other module lists. The manifest hadn't read `boardengine/doc.go`'s Owners section or searched for module lists beyond `main.go`.
3. Read `internal/boardengine/doc.go:170-195` and `internal/machinename/doc.go:1-15`. These are the doc files cards 1 and 2 extend, and the manifest hadn't read their existing text.
