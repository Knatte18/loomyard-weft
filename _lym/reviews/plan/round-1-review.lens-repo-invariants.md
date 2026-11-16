```yaml
lens: repo-invariants
outcome: reported
```

Repo-invariants lens over cards 1–4: seam tests, tier tags, docs the cards make false, and card order.

**Invariants checked, per card**

- **Card 1 (owner-replace-set)**
  - Seam tests: held. No `//lym:seam` marker names `boardengine#OwnerRequest`, so adding `Replace` asks no seam test to change. Its new subtest goes in `TestSetOwners`, whose file is tier1 only.
  - Package `doc.go`: **broken**. Card 1 lists `internal/boardengine/doc.go` only under Extends, but its Intent makes three existing lines in that file false:
    - line 174 says the commit reads "board: owner &lt;machine&gt; for &lt;slugs&gt;";
    - line 183 says that without Takeover the write leaves alone an entry another machine owns;
    - line 185 says only `lym adopt` passes Takeover.
  - `PATTERN.md`: held. The declared `changes` text equals the line at PATTERN.md:24.
  - `pattern/PATTERN-owner-machine.md`: held. Line 27 is covered by the card's Edit. Lines 26 and 37 stay true.
  - Other docs: held. Only PATTERN-owner-machine.md:27 claims that only adopt takes an owner, and the card covers it.
  - Card order: held. This is the first card.
- **Card 2 (machinename-hostname-name)**
  - Seam tests: held, since no struct changes.
  - Package `doc.go`: held. `internal/machinename/doc.go:9` says the package is the only production reader of the hostname, which stays true.
  - Leaf allowlist: held, since `os` is in the standard library.
  - The sole-source hostname scan: held, since the test file is outside the production scan.
  - Card order: held. It needs only symbols that already exist.
- **Card 3 (machine-name-verb)**
  - Tier tags: held. `name_test.go` is tier0 only, and `testmain_test.go` is a support file with no tier.
  - Root wiring: held. `clitree_test.go:152` pins that `newRoot`'s Long text lists every module, and the card adds `machine` there. The registration oracle discovers `Command()` itself. The card adds both seam-signature pins.
  - `PATTERN-cli-cobra`: held. The card adds the `machinecli` Package-naming deviation.
  - Machine-file doc: held. `machinename/doc.go:9` (the package is the only reader of the machine file) stays true, because `machinecli` reads the file through `machinename.ReadFile`.
  - Card order: held. `HostnameName` comes from card 2, and `FilePath` and `ReadFile` already exist.
- **Card 4 (machine-name-sweep)**
  - Tier tags: held. `sweep_integration_test.go` is tier1 only. The new rows stay in tier0 `TestName`, and `TestMain` comes from card 3.
  - Seam tests: held. `Deps` gains fields, but no reflective test walks it.
  - Package `doc.go`: held, since the card edits `machinecli/doc.go`.
  - Card order: held. `Replace` comes from card 1 and the `plan:` handles from card 3.

**Findings**

1. **MEDIUM, consistency.** Location: `_lym/plan/01-owner-replace-set.md`, under `**Extends:** internal/boardengine/doc.go` and the Intent's doc sentence.
   - Summary: card 1 makes lines in `boardengine/doc.go`'s Owners section false and lists that file only to add text to it, so no card rewrites those lines.
   - Evidence: `internal/boardengine/doc.go:174`, `doc.go:183` and `doc.go:185`.
     - Line 174's single commit message no longer holds, because the card gives a replace-set write the subject "board: owner &lt;machine&gt; replacing &lt;names&gt;".
     - Line 183's rule (without Takeover, an entry another machine owns is left alone) no longer holds, because a replace-set write rewrites such entries.
     - Line 185, "Only `lym adopt` passes Takeover", stays literally true. In context, though, it now leaves out the second way past the guard.
   - Fix: move `internal/boardengine/doc.go` from `**Extends:**` to `**Edit:**`. Have the Intent rewrite line 174 to give both commit subjects, and line 183 to except the replace set: it writes recorded owners in the set without Takeover.

**Tool calls**

1. Searched for `//lym:seam` markers on `OwnerRequest` and `machinecli`, and for owner and Takeover lines in `boardengine/doc.go` and the pattern docs. The manifest had not read any seam markers or `boardengine/doc.go`.
2. Searched for a test pinning the root's module list, and for guards on `docs/overview.md`; read `boardengine/doc.go:176-190` and `machinename/doc.go`'s hostname lines. The manifest had not read the clitree pin or either doc.go.
3. Searched for docs that say only adopt moves an owner, to look for falsified doc text outside the cards' targets. That set was not in the manifest.
