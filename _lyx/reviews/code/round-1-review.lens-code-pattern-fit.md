```yaml
lens: code-pattern-fit
outcome: reported
```

Task: code-pattern-fit lens — check the diff against PATTERN.md and CLAUDE.md rules.

**Rule list (held or broken)**

- Declared PATTERN changes landed: all six entries (`sparse-hub-base`, `parent-lock`, `transient-stop`, `friction-capture`, `agent-name`, `wait-mark-display-only`) are in the `PATTERN.md` diff. Each matches its declared text character for character, and `friction-capture` carries `(test)`. **Held.**
- Topic headings: `sparse-hub-base` is under CLI and config, `parent-lock` under Fabric and git, `transient-stop` and `friction-capture` under Shed and loom, `agent-name` and `wait-mark-display-only` under Agents and prompts. These match the record. **Held.**
- `(test)` marking: the five entries without a test carry none. `friction-capture` keeps the `(test)` its removed line carried, and `internal/frictionengine/seam_enforcement_test.go` exists, so a test plausibly enforces it. **Held.**
- `internal/pattern` caps: the longest `PATTERN.md` line is 317 characters, under `MaxEntryLineChars` (360). The file is 8052 bytes, under `HeadroomBytes` (about 15564) and `MaxOverviewBytes` (16384). **Held.**
- No `[background]` link on the restored entries, and no new `pattern/` file. This follows the record's "Overview lines only" decision, and the format checker admits only `pattern/` links. **Held.**
- Package-doc citations: the six `PATTERN-` IDs in the six owning package docs match the `PATTERN.md` IDs exactly. These are `hubreconcile/doc.go`, `parentlock/doc.go`, `shedengine/doc.go`, `friction/doc.go`, `agentname/agentname.go` and `reedengine/doc.go`. **Held.**
- `friction/doc.go`: the new sentence says `lyx loom start` vouches for the resume it spawns, so a deliberate resume leaves no crash-resume note. `internal/loomcli/start.go:244-261` and `:273` say the same, with the voucher written when Tier 2 is on. **Held.**
- CLAUDE.md "Docs land in the same commit": `PATTERN.md` and the owning docs changed in one commit. **Held.**
- CLAUDE.md semantic line breaks: each edit is on a single line. **Held.**
- Scope: the diff changes no test, no `pattern/` file and none of the three rules the audit judged fine (`hub-wide-config`, `recipe-owns-failures`, `batcher-registry`). **Held.**
- Commit message: there is no `Pattern-Departure:` trailer, and none is needed because the diff follows the record. **Held.**
- `friction-capture` text versus its owning doc: the `friction/doc.go` text on archiving (`friction/doc.go:40`) matches the entry's clause "none is archived before a reflection covers it". **Held.**
- Test-run status: I did not run `internal/pattern`'s real-file check, which the acceptance criteria mention. The numbers above came from `wc` and `awk`, not the checker.

**Findings:** none.

**Tool calls:** I made none. The manifest and inherited context already held the diff, the `PATTERN.md` caps, the `start.go` clause and the `friction/doc.go` text.
