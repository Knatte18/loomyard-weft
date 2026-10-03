# Card 17 — specs, overview and CLAUDE.md describe the result

**Prosa:**
- `contracts/specs/webster-spec.md`
- `contracts/specs/loom-plan-spec.md`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `CLAUDE.md`

**Intent:**
Bring the cross-module docs in line with cards 1–16.

- `webster-spec.md`: the verify gate on Merriam's strand, `verify-gate.yaml` replacing `integration.yaml`, the fixer fork and its `fix: <summary>` commits, the "Verify gate fixes" summary section, `verify_gate_attempts`, Merriam's `webster` strand role, and `lyx webster verify`.
- `loom-plan-spec.md`'s verify model: no longer "designed, not implemented".
  Tier 1 is implemented as the Go-derived per-card gate (`go build ./...`, the whole untagged suite, and the integration-tagged tests of the card's own package directories);
  Tier 2 gains that per-card integration-tagged run, while the plan-level `## verify:` stays the once-per-plan run, now a gate on Merriam;
  landing's paragraph says Publish and Finalize verify whenever the tree differs from the last verified tree and halt on a dirty tree.
- `refusal-spec.md`: the integration-regression and integration-report rows go;
  rows land for an exhausted webster verify gate, a fixer-commit `Terminal` failure, a Publish or Finalize dirty tree, a conflict session's untracked file, and the Webster-Burler gate's exhaustion, each with its way forward;
  the parent-write audit row's "integration stage" clause names the verify gate.
- `docs/overview.md`: the webster module entry describes the gate instead of the integration fork, triage and bisect;
  the module tree and the shared-infrastructure list gain `internal/verifytree`, and `internal/verifyrun`'s line names its remaining consumers.
- `CLAUDE.md`'s Merriam entry adds that Merriam's strand is named `<shortname>:<slug>:webster`.
