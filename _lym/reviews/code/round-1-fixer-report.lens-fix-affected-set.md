```yaml
lens: fix-affected-set
outcome: reported
```

No findings. `template.yaml:3` was the only site where the `default_recipe` key's comment lists recipe names, and the fix cleared it.

- **Same-key enumerations:** the remaining `default_recipe` mentions (`config.go`, `board.go`, `deps.go`, `loomcli/doc.go`, `peers.go`, `wire.go`, `boardengine/doc.go`) name no recipes beside the key, except `boardengine/doc.go:206`.
  `boardengine/doc.go:206` names `darn` as the template's value ("`darn` in the template"). That states the value the template sets, which the review's fix allows.
- **Out of scope:** `internal/boardcli/cli.go:497` (`"darn" or "tapestry", or default_recipe`), `SKILL.md:52` and `docs/overview.md:680` list recipe names as the vocabulary for a task's `recipe` field.
  The review's Rejected section already covers `SKILL.md` and `overview.md`; `cli.go:497` is the same kind of site, and the fix did not touch it or make it worse.
- **Other hits:** the broader search for "tapestry, darn or sampler" lists the shared-recipes set in escalate/landing/arbiter docs. None of those is a `default_recipe` site.

Calls made:
1. `grep -rnE "darn, tapestry|tapestry or sampler|..."` over the repo: to find enumerations anywhere else; my context held only the template.
2. `grep -rn "default_recipe"` over the repo, excluding tests: to list every site under the key; my context did not show them.
