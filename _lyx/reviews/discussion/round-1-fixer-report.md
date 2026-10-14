# Fixer report: discussion round 1

## Changes

- F1, fixed as proposed, with one addition.
  The premise holds: reed's Resume relaunches through the stored `ResumeCmd` (`internal/reedengine/strand.go:575`), Claude's resume line is `--resume` on the same session id (`internal/shuttleengine/claudeengine/command.go:211`), a gated run's strand has no done-when list (`internal/shuttleengine/doc.go:166`), and the directive's relaunch paragraphs key on "its Agent call" (`contracts/stencils/twill/twill-directive.md:23`, `:27`).
  D3 now names both ways a session comes back, adds the two resumed-session paths, splits the after-overview path into live (gate re-prompt) and no longer live (liveness finalize evaluates the gate once, so a missing lens review fails `lens-review-missing`), says a resumed session does not count as a new master, and states that the writer-fork part renders these paths in place of the directive's two relaunch paragraphs.
  The addition: a host that asks for no writer fork, Discussion-Write among them, keeps the two paragraphs, since the directive is shared and Discussion-Write would otherwise lose its relaunch rules.
  The Acceptance criteria list each D3 path, the absence of the two paragraphs at Plan-Write, and their presence at Discussion-Write.
- F2, fixed as proposed.
  D6's dispute evidence kinds are now a host value, with the Plan-Write list unchanged and the Discussion-Write list as the fix names.
  A new Acceptance criterion pins that the rendered judge step names each host's kinds, and the Discussion-Write Open risk covers the bound.
- F3, fixed as proposed: the Q4 pick in the support log now matches D3.

## Disputed

None.

## Deferred

None.
