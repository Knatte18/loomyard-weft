# Card 1 — push-timing-config

**Edit:**
- `internal/fabricengine/template.yaml`
- `internal/fabricengine#Config`
- `internal/fabricengine#ConfigSpec`
- `internal/fabricengine#TestConfigTemplate`

**Create:**
- `plan:internal/fabricengine#PushIntervalKey` -> `const PushIntervalKey = "push_interval_s"`
- `plan:internal/fabricengine#PushCeilingKey` -> `const PushCeilingKey = "push_ceiling_s"`
- `plan:internal/fabricengine#PushNoticeAfterKey` -> `const PushNoticeAfterKey = "push_notice_after_min"`
- `plan:internal/fabricengine#PushTiming` -> `type PushTiming struct`
- `plan:internal/fabricengine#Config.PushTiming` -> `func (c Config) PushTiming() (PushTiming, []string)`
- `plan:internal/fabricengine#TestConfig_PushTiming` -> `func TestConfig_PushTiming(t *testing.T)`

**Uses:**
- `internal/configengine#KeyName`

**Intent:** D2's three timing keys, which every later pusher, driver and dobby card reads.
`template.yaml` gains `push_interval_s: 60`, `push_ceiling_s: 300` and `push_notice_after_min: 30`, each with its trailing comment saying what it times.
`Config` gains the three integer fields under those yaml keys, and `ConfigSpec` files each key as `configengine.CategoryMachine`, since a machine on a VPN tunes its own timing.
The keys are new, so no reinterpretation or move table ships.

`PushTiming` carries `Interval`, `Ceiling` and `NoticeAfter` as `time.Duration`.
`Config.PushTiming` converts the three values and returns the warning lines of the values it floored.
A value below one second is floored to one second, per PATTERN-daemon-wakeups, and gives one line naming its key in full through `configengine.KeyName("fabric", <key>)`, per PATTERN-config-key-names.
A ceiling below the interval is raised to the interval, with one line naming `fabric.push_ceiling_s`.
The caller logs each line once at Warn when it loads its timing.

`TestConfig_PushTiming` is a new tier0 table test, adding under a second to one tier0 run.
It pins the defaults read from the shipped template, 60 s, 300 s and 30 min,
and the rows `push_interval_s: 0` floored to one second with exactly one line naming `fabric.push_interval_s`,
and `push_ceiling_s: 10` with a 60 s interval raised to 60 s with one line naming `fabric.push_ceiling_s`.
`TestConfigTemplate` gains the three keys among the template keys it expects.

**ImpactSummary:** Adds three keys and one accessor to fabric's config; no existing reader changes.
