```yaml
lens: leanness
outcome: reported
```

Leanness lens: what in the decision record should move or be cut.

- LOW — `## Decisions` D1–D15, "Rationale:" lines. Several rationales repeat the deliberation, for example D3's "a `beam: true` without a pair would send every member's pair creation…" and D10's "the operator decided…". They pass the completeness test because they carry the why, so keep them. But D6's rationale "folded in from note `fabric-add-adopt-local`, which waited for beams…" is history. It belongs in the support log. Evidence: the spec says Decisions carry Decision + Rationale, and provenance narrative is not rationale.
- MEDIUM — `## Scope`, Out list, the bullet on #724's blocked-run cleanup. It cites the commit history ("landed 2026-10-23", "already ends and removes a run…"). That is research evidence and belongs in the support log's Question ledger, which Q15 already holds. Keep in the record only: "`lyx batten cleanup`'s skip names `lyx batten cancel <slug>`", which D14 already states. So the bullet says the same thing twice.
- MEDIUM — `## Scope`, Out list, "Creating the `workspaces` project and filing its members". That is orch board work, not a task boundary the plan writer acts on. Cut it, or move it to the support log.
- LOW — `## Auto-mode assumptions`. It restates picks that D1, D4, D7–D11 and the ledger already state ("The `order` field name, the `parent_beam` list field and the `P` handle letter"). Prose rule "say it once": shrink it to items that are not already Decisions, or point at the Decisions.
- LOW — `## Constraints`, last bullet ("Docs in the same commit…"). It enumerates doc.go packages, which is a perishable consumer list. The CLAUDE.md rule already applies. Keep only the non-derivable part: `docs/overview.md` for the new module and the `ly:board` skill text.
- LOW — `## Notes for the plan writer`, third bullet ("`orchcli.PrimePaths` and `orchPaths` already split…"). This is a code pointer, and code pointers belong in relevant.md, which already lists both. The second bullet ("check it handles a project entry") is a soft deferred check. Either research it or leave it to Plan; it is not a design fact.
- LOW — `## Open risks`, third bullet. It says a large beam Finalize merge "halts for the orch instead". That is a behaviour claim D11 does not decide, since D11 does not say whether the beam recipe's Finalize declares `on_reroute`. Move it into D11 as a decision ("Finalize carries no on_reroute; a large merge halts"), or cut it.
- LOW — Support log `## Interview`, Turn 1. It repeats the findings that the decision record's Decisions already state. That is acceptable in the support log; no action.

No tool calls made.
