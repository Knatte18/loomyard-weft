<!-- This is the RoleOrchestrator directive: the variant internal/pattern.Directive renders for
     webster's Master session, which only forks and never edits code itself.
     internal/websterengine/render.go is its consuming call site, passing pattern.RoleOrchestrator.
     internal/pattern.Directive reads this file through stencilstore.Read, strips this banner with
     stencil.StripLeadingComment and fills the file's required pattern_overview marker with
     the content of PATTERN.md at the repository's worktree root, and its optional pattern_changes marker with the task's declared PATTERN changes.
     The result is injected as a producer template's optional pattern_directive marker value, so it
     is never itself passed through stencil.Fill again.
     This file declares two markers, the required pattern_overview and the optional pattern_changes, and no other.
lyx-stencil: sha256=fb770dee47107f9a725b9ab19f08be4c38f5e8e614dd7b46ad76d7958c125558 build=6bfaaa1448adcd5fda4ab769115846291406977a time=2026-10-16T12:19:49Z -->

## Constraints — do this before you fork anything

- These are the repo's rules; read them in full before forking a single implementer:

{{.pattern_overview}}
{{.pattern_changes}}

- Read every background file under pattern/ that the entries above link to and that touches what the forks you are about to spawn will do.
- Every fork inherits its context, so reading this once here is what puts the constraints in front of all of them; it must not be skipped on the grounds of not editing code.
- The constraints are BINDING on the forks it spawns: a batch report trading a constraint for a passing verify is a failed batch, not a success.
- A fork that departs from its card because an entry forbids the card's way records it as a `Pattern-Departure:` trailer in its commit, as `loom-plan-spec.md` defines, and the digest's `departures` carries it.
