```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. I list, per change, what the change forces outside its own lines, and whether the diff covers it.

**Per change**

- **`Request.ResumeOnWake` (`request.json`, key `resume_on_wake`).** Set found: `Submit`, `Watch`, `failAtWake`, `Status`, `internal/hibernateengine/doc.go`'s request-file paragraph, and the fixture `fakeWorld.ownRequest`, which now seeds `ResumeOnWake: true`. `Status` returns the whole `Request`, so the key shows in `lym hibernate status` with no code change. Covered.
  - A request file written before this change reads as false, so a new daemon keeps the pause. `doc.go` says writes keep keys a newer binary added, and the summary documents the old-file reading. Covered.
- **The wake now lifts the pause only on `ResumeOnWake`.** Prose that described the lift-always rule:
  - `internal/hibernateengine/doc.go` (the Cancel and wake paragraphs): updated.
  - `internal/hibernatecli/cli.go` (the `request` and `cancel` Long text): updated.
  - `internal/hibernatecli/doc.go`: updated.
  - `docs/overview.md` (the package tree line and the hibernate module bullet): updated.
  - `contracts/orchrole/orch-role.md` (the hibernate lines): updated.
  - `watch.go` (the file header and the `Watch` godoc): updated.
  - My grep over `internal`, `contracts`, `docs/overview.md`, `pattern` and `cmd` for hibernate with lift, resume or wake found no other stale wording.
  - The remaining hits are `pauserecord/doc.go:18`, `tendercli/doc.go:36`, `cleanupcli/reconcile.go` and `cleanupcli/doc.go:27`. They concern the owner-pause lift, `EndOwnedElsewhere` and pair-level wake holds, none of which the change touches. Covered.
- **`Cancel` refusal message.** The only other thing that restates it is the `cancel` Long text, which is updated. Covered.
- **New hub-wide module `hibernate` (`internal/hibernateconfig`).**
  - `configreg.Modules` has the entry.
  - `TestModules_SeedOnlyAndHubWideFlags` carries the new name, in both its comment and its `wantHubWide` expression.
  - `configstrictness_test.go`'s degrading set and `pattern/PATTERN-config-layers.md`'s degrading-callers list both add `hibernateconfig`.
  - `ConfigSpec` classifies the one key as cross-pair, which `TestModules_EveryTemplateKeyClassified` requires, and the template carries it.
  - `docs/overview.md` lists the package.
  - A grep for other module lists (`language.yaml`, `sampler.yaml`, `heddle.yaml`, `"sampler"`) found none that enumerates hub-wide modules, so no registry or doc list is left without it. Covered.
- **`configcli.go` `buildConfigLong` edit.** It drops the "which lists the modules and edits the one you choose" clause. Commit `74a85d66` is a fix to keep the Long page under the 2000-byte cap, which the added module's listing pushes it past. The edit is licensed by that commit and rides along for a reason. It is not a finding.
- **PATTERN declared changes.** The directive declares none. The `pattern/PATTERN-config-layers.md` edit only adds `hibernateconfig` to the degrading list. No `Pattern-Departure:` trailer appears in the three commit messages, and none is needed: the constraint checks I could apply here (`PATTERN-config-layers`, `PATTERN-stencil-ownership`) show no violation.
- **Stencil wording.** The only stencil-like text changed is the orch role's hibernate lines. They keep the existing instructions (`--after`, `--after-all`, `status`, `cancel`) and add the flag guidance. Nothing is dropped or weakened.

**Findings:** none. Every member of the affected set that I found is updated and agrees with the behaviour.

**Tool calls (3 of 6)**
1. `grep` for hub-wide module name lists (`heddle.yaml`, `sampler.yaml`, `language.yaml`): my context lacked any search for other module enumerations.
2. `grep` for hibernate with lift, resume or wake outside the hibernate packages, first attempt: it failed with a regex complexity error and returned nothing.
3. The same search with a simpler pipeline: my context lacked other docs, specs and pattern files that restate the wake-lift rule.
