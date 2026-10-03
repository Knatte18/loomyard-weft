# Card 1 — testkit-root

**Create:**
- `testkit/doc.go`
- `testkit/enforcement_test.go`

**Edit:**
- `//CONSTRAINTS.md`
- `//docs/overview.md`

**Intent:**
Reverse the "duplicate test helpers rather than share them" policy for seams faked in two or more packages, and land its replacement, the Testkit Invariant (D1, D3), before any kit exists.

Before making any change, record the baseline leaf test count (see the overview's Shared Decisions) and put it in this card's commit body.

`internal/testkit/doc.go` is a doc-only `package testkit`: it states that shared test kits live one package per kit under `internal/testkit/<kit>/`, never under the package they fake, that a kit imports only the lowest packages defining the types it fakes, that a fixture used by one package stays in that package's `_test.go` files, and that a package an import cycle bars from a kit keeps exactly one local copy.

`internal/testkit/enforcement_test.go` (untagged, `package testkit`) is one AST scan modelled on `internal/shedrecipe/seam_enforcement_test.go`, resolving the module root from `runtime.Caller` so it spawns nothing and needs no Tier Purity allowlist entry.
It enforces three rules, each reported by name:

- Importer rule: walking `internal/`, `cmd/` and `tools/`, no non-`_test.go` file outside `internal/testkit/` imports a path under `internal/testkit/`.
  Files under `internal/testkit/` are exempt so a kit may build on another kit (an `envkit` builder filling seams with `shedfake` fakes); every kit is itself reachable only from tests.
- `*cli` rule: no non-test file under `internal/testkit/` imports an `internal/*cli` package.
- Spawn-import rule: no non-test file under `internal/testkit/` imports `os/exec`, `internal/gitexec`, `internal/gitkit` or `internal/hubforge`; `internal/testkit/lyxbin` is the one exemption, and only from `os/exec`.

It carries a vacuous-scan floor: the importer walk must have parsed at least one file.
The scan's own shape is not final (part 2 may rebuild it on its harness), so keep it to these rules with no extension points.

`CONSTRAINTS.md` gains a "Testkit Invariant" section stating the D3 rules, the kit-on-kit exemption above, the `lyxbin` exemption bounded to `go build` of `./cmd/lyx`, and which parts are review discipline: a kit starts no process, tmux server or agent beyond what its imports allow, and asserts nothing beyond `t.Fatalf` on its own setup, the `shedfake` `Call`/`RequireOutcome` outcome check and the `envelope` `RequireOK`/`RequireErr` shape check (the two named helper pairs #324 specifies).
`docs/overview.md`'s test-infrastructure passage beside `gitkit` and `hubforge` gains one paragraph pointing at `internal/testkit` and the invariant, without listing kits.

Prove the scan by planting one violation of each rule in turn: a non-test importer, a `*cli` import in a kit, an `os/exec` import in a kit other than `lyxbin`, and a `gitkit` import in a kit.
Watch each fail, then remove every plant; nothing planted is committed.

What this lets through: a shared fake now misleads every package that uses it, bounded by the kit rule that a fake reproduces the richest copy it replaces.

**ImpactSummary:** Adds one doc-only package with its scan and one new CONSTRAINTS.md section; no existing code changes.
