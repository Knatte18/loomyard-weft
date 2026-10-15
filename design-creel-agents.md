# creel stage 3: agents message through creel, not SendMessage

`creel-agents` · Task · feature, shuttle, loom, stencil

Depends on: [`creel-notices`](design-creel-notices.md), [`engine-config-seams`](design-engine-config-seams.md)

## Goal

Creel stage 3: agents reach their parent and each other through creel, not Claude Code's `SendMessage`, so a non-Claude engine can take a role that has a parent.
The design is in `creel-core`'s body until that task lands, then in the creel package's `doc.go`.

## Operator decision (2026-10-15)

Creel is a prerequisite for Codex; the Codex-readiness plan's interim `neutral-parent-asks` is dropped in favour of this stage.

## Scope (stage 3 of the creel design, plus the Codex-readiness findings)

- The parent-review gate (`internal/parentreview/gate.go`, which today prompts Discussion-Write to `SendMessage` the parent) posts itself.
- Drivers' stop reports post instead of duplicating batten's notices over `SendMessage`.
- `lyx shuttle send` and `lyx shuttle interrupt` move behind creel.
- The parent directive (`contracts/stencils/parent/parent-directive-parent.md`), rendered for every role with a parent, says to ask through creel, not "by `SendMessage`".
- `shuttleengine/hold.go`'s hold notice names creel, not `SendMessage`.
- The other stencils that name `SendMessage` (`seat-directive-{chair,advisor}`, `shed-template-driver-notify{,-watched}.md`, `loom-template-parent-review-delivery.md`, the orch role) move to creel where the Discussion finds it fits; what stays Claude-only is declared through `Spec.Requires`.

## Risk to check in Discussion

The hold notice goes to the prime orch (`orchcli.NotifyPrime`), while the parent directive names `parent_name`, which may be a loom driver strand: check per recipe that a question reaches the session that answers it.
