# Card 9 — landing-idle-config

**Create:**
- `plan:internal/landingshed#defaultIdleReleaseMin` -> `const defaultIdleReleaseMin = 10`
- `plan:internal/landingshed#Config.IdleRelease` -> `func (c Config) IdleRelease() time.Duration`
- `plan:internal/landingshed#TestConfig_IdleRelease` -> `func TestConfig_IdleRelease(t *testing.T)`

**Edit:**
- `internal/landingshed#Config`
- `internal/landingshed/template.yaml`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the parent lock's idle bound to the hub-wide `landing.yaml` (D7, `PATTERN-hub-wide-config`).
`Config` gains `ParentLockIdleReleaseMin int` under `parent_lock_idle_release_min`, and `template.yaml` gains the key with the value 10 and a comment saying it is the minutes a waiting run, or an undecided PR-Gate hand-off, keeps its parent lock before the lock is released, and that a value below 1 reads as 10.
`IdleRelease()` returns the value in minutes as a duration, and a value below 1, an absent key included, returns `defaultIdleReleaseMin` minutes with one Warn naming the key.
The template change alters the config registry's fingerprint, so `hubreconcile.Ensure` adds the key to existing hubs on the next binary change (`PATTERN-binary-change-reconcile`); nothing else writes it.
`TestLoadConfig_TemplateDefaults` asserts the template default; `TestConfig_IdleRelease` is one table over 10, 3, 0 and -1.
`TestConfig_IdleRelease` lives in `config_test.go`.

**ImpactSummary:** one new hub-wide landing key with a reader; nothing reads it yet.

**Commit:** 9: landing: parent_lock_idle_release_min
