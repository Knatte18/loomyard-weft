```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set: what the `docs/overview.md` merge forces outside the lines it touches. The result is no findings.**

Per change, the set I found:
- **Board entry, `docs/overview.md:358`:** "prints the hub's profiles with their settings and each recipe's default profile". It agrees with `internal/boardcli/doc.go`, which describes the `defaults` field. Covered.
- **Config entry, `docs/overview.md:372-373`:** it says Seed-Child writes the task's recipe block, a task naming none takes its recipe's default, the starting profiles are `speed`, `light`, `solid` and `wide` plus the darn-only `docs`, and `--set` replaces a whole profile. It agrees with `internal/runprofile/doc.go`, `template.yaml` and `internal/battenshed/doc.go`. Covered.
- **Gate entry, `docs/overview.md:461`:** only the parent changed it. The merge kept the parent's text intact, and it has no profile content. Covered.
- **Module-table line, `docs/overview.md:313`:** "template, layered-load spec, loader and lookup" is still accurate. Covered.
- **Old profile names:** a grep of `docs/` (excluding `docs/research`), `README.md`, `plugins/` and `contracts/` for `darn-wide`, `budget` next to "profile", and `profiles.yaml` found nothing stale. The only `profiles.yaml` hits are the two overview lines above. Covered.
- **Parent changes since the settle:** the parent changed none of `internal/runprofile`, `boardcli`, `configreg`, `battenshed`, `battencli`, `configcli`, `boardengine`, `plugins/ly/skills/board` or `contracts/stencils/orch` between `04aafa88c` and `c9c4e7ddf`. Covered.

Tool calls:
1. Grep of `docs/`, `README.md`, `plugins/` and `contracts/` for the old profile names and `profiles.yaml`. My context held no cross-doc search.
