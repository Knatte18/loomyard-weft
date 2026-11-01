No findings.

- F1: the stencil search found no other file that tells the orch to poll runs; the `batten status` mentions at `orch-role.md` lines 117 and 123 are stop investigation, which the new rule allows.
  Noted, not rated: `internal/orchengine/doc.go:433` still says the line tells the orch to "check each run"; `internal/orchcli/refusal.go:44` is operator-facing text.
- F3: `orch-role.md:269` is the only `cd <` or `&& lym` site in the role and the orch stencils, and it now says the command is for the operator to type.
- F4: the new assertion checks both `Names()` and `Default()` inside the existing test; nothing else needs covering.

Tool calls: two.
1. grep for `check each run` and `batten status` under `contracts/`, `internal/orchengine` and `internal/orchcli`, because the context did not show the other sites.
2. grep for `cd <` and `&& lym` in the role and the orch stencils, because the context did not show whether another `cd` instruction exists.
