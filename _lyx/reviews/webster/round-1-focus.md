---
round: 1
focus:
  - "Cards 3 and 4 (`standingDenies` rows in `claudeengine`): check each row against the `deny-row-shape` decision, namely a steer and notice free of quotes, backslashes and line breaks, and a row comment that states the bound as a guardrail, not a barrier. Settle whether the hook and system-prompt notice still derive from the one table."
  - "Card 3 in-place deny: settle whether `sed -i`, `sed --in-place`, combined short flags such as `-ni` or `-i.bak`, and `perl -i`/`-pi` are all refused, and whether read-only `sed -n` is never refused."
  - "Card 4 computed-rm deny: settle whether `rm` with a substitution or variable expansion is refused and plain `rm` of a literal path is not, including `rm` reached through a pipe, `&&` or `xargs`."
  - "Card 6 sed print classifier: settle whether quiet-flag print scripts with regex addresses are admitted and whether any script with a write, execute or in-place form (`w`, `e`, `-i`, `-s` with `w`) is still refused."
  - "Card 5: settle whether `sed -i` and `perl -i` join the no-denied-recovery closed list consistently with the denies of cards 3 and 4, with no form that a deny refuses left recoverable by another route."
  - "Card 7 dialog notice: settle whether the parent is notified once per dialog (not per poll), only after the 60 s bound, only for autonomous runs, and whether the bound and the once-only state are tested."
  - "Cards 1 and 2: settle whether the edit directive wording matches the denies' actual scope and whether `.scratch/` is excluded in each warp repo's `info/exclude` idempotently, without duplicate lines on rewiring."
  - "Test economy: for each new test, check it against existing coverage in the same package and against the `cmd/lyx/testdata/test-budget.yaml` row; name the covering test for any redundancy finding."
  - "Docs landing: settle whether the diff updates the affected `doc.go`, `docs/overview.md` and `PATTERN.md` where an observable behavior or cross-cutting invariant changed."
---

Seed focus drawn from the plan's card index and its one shared decision; no prior round exists.
