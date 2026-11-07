```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over `internal/hubforge/template.go` and its callers in `hub_test.go`. Each entry is the case I ran and whether it is correct.

**Per function or branch**

- `slotFor`, same registration twice: the slot is found and the pointers match, so it is accepted. Correct.
- `slotFor`, two registrations with one key: the second caller's `Fatalf` names the key. The map lock is released before the compare, and `slot.registration` is never written after creation, so there is no race. Correct.
- `slotFor`, a user key equal to a shape name (`.+pair`): whichever registration claims the slot first wins, and the other fails naming the key. Correct, and matches `summary.md`.
- `templateFor`, two parallel first callers of one key: they serialise on `slot.mu`. The second sees `template != nil` or `failed`, so the build runs once. Correct.
- `templateFor`, build fails: `failed` is set before `Fatalf`, and the deferred unlock runs on `Goexit`. The first caller gets the build's errors joined under the key. A later caller hits the `failed` branch and names the key without rebuilding. Correct.
- `templateFor`, two keys: only the slot's own mutex is held across the build. The map lock covers just the lookup and insert, so different keys build at once and the map lock is never taken in inverted order. Correct.
- `buildTemplate`, the `template` variable: it is written in the goroutine and read after `<-done`, so the channel close orders the accesses. `FailNow`, `Fatal`, `Fatalf` and the refusals all end in `Goexit`, and `defer close(done)` still runs. Correct.
- `buildTemplateIn`, any early `Fatalf`: `markFixtureBuild`'s release runs via defer, so the ref count stays balanced. The half-built root is left in the lymtemp run directory, which `tmuxkit.Main` removes. Correct.
- `buildTemplateIn`, a non-fatal `Errorf` or `Fail` in the build: the build continues through the tip check and fingerprint. `buildTemplate` then returns the template with failures, and `templateFor` marks the slot failed, so the partial template is never published. Correct.
- `requireBoardAtUpstreamTip`, board ahead of the bare: HEAD differs from the bare's tip, so it fails.
- `requireBoardAtUpstreamTip`, a commit pushed from `root/pusher` into the bare: the tip is read from the bare, not the tracking ref, so HEAD differs and it fails.
- `requireBoardAtUpstreamTip`, an unpushed pair or other branches: only the board's branch is compared. Correct per D2.
- `requireBoardAtUpstreamTip`, detached HEAD: `CurrentBranch` returns `HEAD`, the `rev-parse` of `refs/heads/HEAD` fails inside `gitkit.Git`, and the build fails. This is loud and `templateFor` prefixes the key.
- `templateBuildTB` refusals: `Cleanup`, `Setenv`, `TempDir`, `Chdir`, `ArtifactDir`, `Skip`, `Skipf` and `SkipNow` all record, then `Goexit`. The tests cover the five spec'd calls. Correct.
- `templateBuildTB`, the pass-through methods:
  - `Log`, `Logf`, `Name` and `Skipped` forward to the first caller's TB, which is blocked in `<-done`. The calling TB is therefore still live and nothing is touched after it ends.
  - `Context()` passes through, as D2's Bound section acknowledges.
  - `Parallel` is not part of `testing.TB`.
  - Correct.
- Shape builds under the refusing TB: the build path uses `AddPair`, `cloneHub`, `forgetCloneCorrespondence` and the `gitkit` helpers. None of them calls the refused methods. My grep of `gitkit/*.go` found only `tb.TempDir()` in the clone-a-bare helper, at `gitkit.go:341`, which the build does not use. Correct.
- `CopyHub`, `SharedHub` and `hubAt`: relocation, rebase and fingerprint are unchanged. `hubAt` now receives the template instead of re-resolving it, with no behavioural change. `shapeRegistration` resolves the closed set and `Fatalf`s on a shape outside it. Correct.
- `copyTemplate`, `requireIntact` failure message: it says "take CopyHub in the test that writes", but for a registered key the call to take is `CopyRegistered`. The message is misleading, though it still names the fixture.
- `commitRelocatedBinding`, a build with several commits: the amend folds the binding into HEAD only and the force-push sends HEAD. That is safe only because the tip check guarantees the bare holds nothing beyond HEAD. Correct.

**Findings**

1. **LOW, design, `template.go:177-187`.** `buildTemplate` runs the build on a goroutine with no `recover`. A panic in a `Build` function, or in a helper it calls, is a panic on a goroutine the test framework did not start. It aborts the whole test binary, so the slot is never marked failed and no later caller gets a key-naming failure. This is loud, and the same panic in an ordinary test also aborts the binary. But the focus asks for failed-marking "including when the build panics", and D2 says a failed build marks the template failed.

2. **LOW, design, `template.go:166-172`.** A `Build` that exits its goroutine without a recorded failure leaves `failures` empty and `template == nil`. One way is a direct `runtime.Goexit()` or a `Goexit` from a helper that bypasses the TB. `templateFor` then stores and returns a nil template, so `copyTemplate` panics on a nil dereference at `requireIntact`. All the TB-routed exits record first. This is unlikely and arguably internal misuse, so I list it only for completeness.

3. **NIT, design, `template.go:356`.** `requireIntact` tells the user to "take CopyHub in the test that writes", which is wrong advice for a registered key.

I found no further defects in the lock structure, failure marking, identity check, TB refusals, tip check or build order.

**Tool calls**
1. `grep` in `internal/gitkit/*.go` for refused TB methods, since the manifest did not include `gitkit.go`; this checks that the shape and registered build paths never call a refused method.
