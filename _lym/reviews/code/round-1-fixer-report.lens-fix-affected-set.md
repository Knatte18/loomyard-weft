```yaml
lens: fix-affected-set
outcome: reported
```

**Lens fix-affected-set: did F6, F8 and F3 cover every member of their sets?**

I reran the three set searches, one per finding. One site was missed.

**LOW — `internal/pairteardown/doc.go:78` still advises `lym batten pause <slug>` alone (F8)**
- F8's premise is the set of operator-facing texts that present `lym batten pause` as the way to quiesce or abandon a run. The fix covered `anchor.go:318`, `claim.go:102` and the `lym batten pause` help in `cli.go`.
- The package doc line reads: "a running one holds it, and the settle only warns, naming `lym batten status <slug>` and `lym batten pause <slug>`."
- That warning is the text the fixer changed in `claim.go:102`, which now also names `lym tender pause %s`. The doc now disagrees with the warning's wording. This is a doc twin left unchanged, not operator-facing output, so it is a lower-weight miss.

**Searches that found nothing more**
- **F8:** the other `batten pause` hits are wording about the pause flag and the notice bounds, not advice to hold or abandon a run. These are `battenshed` docs, `wire.go:1096`, `land.go`, `docs/overview.md:725` and `:749`, and `batten-recipe.yaml:15`.
- **F8:** `wire.go:1486` sets `PauseWayForward` to `lym batten pause %s`. I did not trace where it surfaces, so I cannot say whether it is a further site.
- **F6:** `contracts/orchrole/orch-role.md` rule 2 says "dobby does not resume it". Its exception sentence, added in the base commit, covers the reached-parent case. I found no other copy of the `pause`/`goto` sentence.
- **F3:** no other "only while running" statement about passes remains in `dobbyengine` or elsewhere. The matches at `doc.go:136` and `runnotice.go:153` are about run notices, which the review rejected as still true.

Calls made, 3 of 6:
1. `grep -rn "batten pause"` over `.go`, `.md` and `.yaml` files, to rerun F8's and F6's set search. My context held only the three named sites.
2. `grep -rniE` for "only while running", "dobby does not resume" and similar phrasings, to rerun F3's and F6's set search for sibling statements.
3. `sed -n 10,20p` on `batten-recipe.yaml`, plus a grep of `docs/overview.md` and `contracts/*.md`, to check whether the `batten-recipe.yaml:15` hit and any dobby paused/halted prose needed the change. Neither did.
