```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Findings I checked by rerunning their searches: F8, F1 and F5; F15 was only partly searched.

- **F8:** I searched the prose lists naming recipes. The two edited places (the `Run-Shed` sentence in `docs/overview.md`, the inner-run sentence in `contracts/orchrole/orch-role.md`) are the only lists of startable recipes. The other matches are lists of recipes with a landing-escalate halt, a `Describe` row or a loom perch.
  - `docs/overview.md:1146` ("a recipe declaring `contract: loom` is served, among them tapestry, … sampler") and `docs/research/loom/lym-state-map-2026-11-15.md:318` do omit hardener, but the review did not name them and I could not tell whether hardener belongs in them.
  - `contracts/orchrole/role-beams.md:73` ("a task run's is a tapestry, motif, darn or sampler row") is the one hit whose wording could plausibly want hardener. Whether hardener runs in a beam is outside what my context shows.
- **F1:** `ArchiveDeps` is built only in `internal/hardenshed`, and no other package references `ArchiveDeps` or `.Archive`. All `Campaign{` constructors (`internal/loomcli/hardenwiring.go`, `internal/testkit/envkit/envkit.go`, `internal/shedrecipe/entries_harden_test.go`) now set `AnchorPath`.
- **F5:** The template default `fixer: [opus, fable, sonnet]` in `internal/hardenshed/template.yaml` is non-empty, so Preflight over default config still passes. Only a `run_settings` or local override can empty the list, which is the case the review described.
- **F15:** The earlier edit lists Harden-Burler-Fix in the `PATTERN-gate-slot.md` Verify-sites sentence. `pattern/PATTERN-test-speed.md:57-58` already names both its gates.
- **F7, F11:** Not searched; both are single call sites, `runWithSlots` and `greenForCall`, which the fix covers.

Calls made:
1. Searched prose, Go and YAML for recipe-name lists to see which lists the F8 fix left without hardener. My context had only the two lines the review named.
2. Read `internal/hardenshed/template.yaml` and searched for `ArchiveDeps` consumers outside `internal/hardenshed`. My context lacked the fixer default and the full set of F1 consumers.
3. Searched the markdown for `Harden-Burler-Fix` mentions to check the F15 set. My context did not show the other pattern files.
