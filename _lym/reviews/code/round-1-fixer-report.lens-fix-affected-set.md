```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the set F1's fix had to cover, the consumers of the landing-notice follow-up, beyond `role-beams.md`.

No findings.

The search for `beam describe` prompts found one landing-notice follow-up site, `internal/dobbyengine/landnotice.go:165-181`. It is gated by the same `land_notice` rule the fix describes. `role-beams.md:55-56` is the other site and is already named in the new blast radius. The other hits (`beamcli` refusals, `role-autonomy.md:56`, `docs/overview.md:838`) are unconditional prompts that don't depend on `land_notice`, so they are not affected.

Unchecked: the `member` key being absent from the state record. I did not rerun that search, since the review makes it a format claim, not a set.

1. Bash `grep -rn "beam describe"` over `*.md` and `*.go` outside `_lym`, `.lym` and tests. My context held no listing of the other sites that tell an orch to run `lym beam describe`.
