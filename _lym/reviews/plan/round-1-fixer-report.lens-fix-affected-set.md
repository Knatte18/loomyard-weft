```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I re-ran the set searches for the set-premise findings and checked that the fix covers every member.

Unchecked findings: F2 (I did not re-scan `hardenshed`/`hardencli` code for side names, because those packages don't exist yet), F8, F11, F14.

- **LOW.** Card 2's changed text adds rows to `internal/gateslot/doc.go`'s site table and to `PATTERN-test-speed.md`'s key table, but it leaves a third site list in the same set untouched: `pattern/PATTERN-gate-slot.md:104-105` ("Code-Burler-Fix and Darn-Write each run their own gate key's call through `Verify`: the `code-burler-fix` and `darn-write` keys"). Harden-Burler-Fix's `verify` gate now runs its own key, `harden-burler-fix`, by the same route. `internal/loomshed/doc.go:113-114` names those same two keys per row in the same way, and card 13 lists that file only under Extends. Evidence: the grep for `code-burler-fix` across docs and pattern found both lines, and no card's Edit list names `PATTERN-gate-slot.md`.

Other sets were clean:
- **F7:** the remaining `code-burler-fix` hits are not enumerations of the template's `diff` keys: `gateslot/doc.go:238` is the migration, `gateslot/config.go:596` is the landing group, and the others are examples or research records.
- **F15:** the only task-recipe lists are `boardengine/task.go:351` and `boardcli/cli.go:604`, and both are covered. The other "tapestry, darn or sampler" texts list landing or escalate recipes, not task recipes.
- **F12:** the other treadle mentions are covered elsewhere. `shuttleparity` and `cmd/lym#pathRules` are in card 18, `verifyrun` and `agentconfig` in card 19, and card 20 leaves `docs/benchmarks` and `docs/research` as point-in-time records.

Tool calls:
1. Grepped `code-burler-fix` across sources and docs, to rerun F7's search for sites that enumerate the keys.
2. Grepped the task-recipe enumerations, to rerun F15's search.
3. Listed the files naming treadle, to rerun F12's search.
4. Checked cards 18 and 20 for the remaining treadle files and read the `PATTERN-gate-slot` and `loomshed` doc spans, because my context held neither those cards nor those lines.
5. Checked card 18 for `cmd/lym` and every card for `PATTERN-gate-slot`, to confirm coverage.
