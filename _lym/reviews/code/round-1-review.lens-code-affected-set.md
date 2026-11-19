```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. For each change in the diff, I traced what it touches outside the changed lines.

**Change by change: what it reaches, and whether that is covered**

- **`Read` now returns the shipped default for an untouched, newer-written copy** (`internal/stencilstore/reconcile.go:31`):
  - Its own doc comment: updated, covered.
  - Package doc `internal/stencilstore/doc.go`: updated, covered.
  - The rule `PATTERN-stencil-ownership` (`PATTERN.md:41`, `pattern/PATTERN-stencil-ownership.md:7,12-13`) says prompts are "never embedded" and "None is read from embedded bytes": not covered. See F1.
  - Commit trailers: neither commit in the range carries a `Pattern-Departure:` trailer. Not covered, see F1.
  - The comment in `internal/shedadapters/bouncer.go:330`, "stencilstore.Read never falls back to a shipped default": now false, not covered. See F2.
  - `internal/editdirective/doc.go:43` and `internal/twilldirective/doc.go:51`, "It is never read from embedded bytes": now false in the skew case, not covered. See F3.
  - The `Registry()` doc at `contracts/stencils/stencils.go:1134`, "stencilstore.Read, which needs no registry": now stale, not covered. See F4.
  - The pins notice in `internal/loomcli/stencilpins.go` (`stencilsChangedNotice`) tells the parent that later rows "render prompts that differ from its earlier ones". A changed pin whose copy is untouched and newer-written now renders the shipped default, so the claim can be wrong. Not covered, see F5.
  - Tests that call `Read`, `ReadSet` or `ReadForEngine` in the `stencilstore` package run with no registry registered, so they behave as before. Other packages' test binaries carry no VCS time, so the fallback stays off there. Covered, nothing to fix.
- **`readInclude` looks up engine blocks under the key `EngineBlockName(engine, block)`** (`internal/stencilstore/engineblock.go:164`):
  - The registry's `Default` looks entries up by name. The embedded entries are under `engine/&lt;engine&gt;/&lt;block&gt;` (`contracts/stencils/stencils.go:702-756` embeds), which is the same form `EngineBlockName` produces. Covered.
- **`init` in `contracts/stencils` calls `RegisterShipped`**:
  - The production importers (`cmd/lym/stencilseed.go`, `cliwire`, `loomcli`, `stencilcli`) all link the same single `init`. Covered.
  - `internal/testkit/stencilkit` imports it as well, and has no effect at test time because the running time is zero. Covered.
- **New package doc paragraph in `doc.go`**: matches the code. Covered.
- **Declared PATTERN changes**: the directive declares none, so there is nothing to check.

**Findings**

F1
- Severity: BLOCKING
- Class: design
- Location: `internal/stencilstore/skew.go:59-71`, reached from `reconcile.go:31` and `engineblock.go:164`
- Summary: `Read` now returns a prompt from embedded bytes (`registry.Default`). That breaks `PATTERN-stencil-ownership`, which allows only the orch role to render from embedded bytes. The range has no `Pattern-Departure:` trailer and no PATTERN edit.
- Evidence: `pattern/PATTERN-stencil-ownership.md:9,12-13` and `PATTERN.md:41`. `git log` over the range shows no trailer.
- Fix direction: the Sketch's other option needs no departure — fail early with an error that names the skew and the way forward. Otherwise, a recorded departure plus a PATTERN amendment.

F2
- Severity: LOW
- Class: consistency
- Location: `internal/shedadapters/bouncer.go:330`
- Summary: the comment "stencilstore.Read never falls back to a shipped default" is now false. Only the missing-file case still holds, so the comment should say that.
- Evidence: `internal/shedadapters/bouncer.go:330`, `reconcile.go:28-29`.

F3
- Severity: LOW
- Class: consistency
- Location: `internal/editdirective/doc.go:43` and `internal/twilldirective/doc.go:51`
- Summary: both package docs say their stencil "is never read from embedded bytes", and they read it through `stencilstore.Read` and `ReadForEngine`, which can now return embedded bytes.
- Evidence: those lines; `skew.go:130`.

F4
- Severity: LOW
- Class: consistency
- Location: `contracts/stencils/stencils.go:1133-1134`
- Summary: the `Registry()` doc says `stencilstore.Read` "needs no registry", but `Read` now depends on the registry this package's `init` registers.
- Evidence: `stencils.go:1134`, `skew.go:99-103`.

F5
- Severity: LOW
- Class: consistency
- Location: `internal/loomcli/stencilpins.go`, `stencilsChangedNotice`
- Summary: the notice says the changed stencils make later rows render differing prompts. For an untouched copy written by a newer binary, the running driver now renders its own shipped default, so the notice overstates what changes.
- Evidence: the notice text in `stencilpins.go`, `skew.go:118-131`.

**Tool calls**

1. A grep for "never falls back" / "embedded bytes" / "needs no registry" across `.go` and `.md` files, to find docs and comments outside the diff that describe `Read`'s old behaviour. My context had none of these.
2. A read of `bouncer.go:320-340`, `editdirective/doc.go`, `twilldirective/doc.go` and the `PATTERN.md` line, to confirm the exact wording of the hits from call 1.
