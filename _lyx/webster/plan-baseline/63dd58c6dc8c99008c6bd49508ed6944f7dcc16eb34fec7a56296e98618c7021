# Card 13 — role-prefix-config

**Create:**
- `internal/shedrecipe#rolePrefixKey`
- `internal/shedadapters#Bouncer.role`
- `internal/burlerengine#Profile.rolePrefix`

**Edit:**
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe#TestBouncerEntry_EnvJudgeFallback`
- `internal/shedrecipe#TestBouncerEntry_ConstructionFailures`
- `internal/shedrecipe#TestBurlerHalfEntries_ColorSegment`
- `internal/shedrecipe/doc.go`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#bouncerJudgeRole`
- `internal/shedadapters#bouncerSeedRole`
- `internal/shedadapters#Bouncer.awaitLiveJudge`
- `internal/shedadapters#Bouncer.awaitLiveSeed`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedadapters#TestBouncer_ReBounceProbesForALiveSeed`
- `internal/shedadapters#TestBouncer_EntryProbe_SpecNamesTheJudgesOwnOutputFiles`
- `internal/shedadapters/doc.go`
- `internal/burlerengine#Profile`
- `internal/burlerengine#Profile.reviewTable`
- `internal/burlerengine#Profile.fixTable`
- `internal/burlerengine#fakeShuttle.scriptFor`
- `internal/burlerengine#TestEngine_Review_TableShape`
- `internal/burlerengine#TestEngine_Fix_TableShapeAndFiles`
- `internal/burlerengine/doc.go`

**Intent:**
The Bouncer, BurlerReview and BurlerFix rows take an optional recipe config key `role_prefix` (D8).
`rolePrefixKey`, in `entries_bouncer.go`, reads it with `configString` and refuses a present value `agentname.ValidateRole` rejects, with an error naming the entry, the key and the value and the way forward (a lowercase `[a-z][a-z0-9-]*` prefix such as `plan`); an absent key gives "".
`bouncerEntry`, `burlerReviewEntry` and `burlerFixEntry` each call it and add `role_prefix` to their `configRejectUnknown` lists, so the refusal fires when the row is built, before any spawn.
The key changes only the role strings a row's sessions carry: the spawn color stays on the launch spec's segment (PATTERN-spawn-color), and only `agentname`, through reed, forms the strand name from the role (PATTERN-agent-name), so the strand, window and bar button names carry the prefix with no reed change.

Bouncer: `BouncerConfig` gains `RolePrefix`, which `bouncerEntry` fills.
`Bouncer.role` returns `<prefix>-<base>` for a non-empty prefix and `base` for an empty one, and every place that names a seed or judge role passes through it: the judge spawn in `judgeCall`, the seed spawn in `runSeedSpawn`, and the live probes `awaitLiveJudge` and `awaitLiveSeed`, so a spawn and its probe always carry the same role.
With the key set the sessions carry `<prefix>-bouncer-seed` and `<prefix>-bouncer-judge`; with it empty, today's roles.
The constants' and `awaitLiveSeed`'s comments claim Attach matches on the role; shuttle's attach matches on output files, so a session spawned under the old role is still attached, and the comments are corrected to say the role is an identity field kept equal for attribution.

Burler: `Profile` gains `RolePrefix`, which both burler entries fill, the path the color segment already takes.
`Profile.rolePrefix` joins it to a base as `Bouncer.role` does, and `reviewTable` and `fixTable` set the seat table's `RolePrefix` to `rolePrefix(reviewRolePrefix)` and `rolePrefix(fixRolePrefix)`, giving `<prefix>-burler-review` and `<prefix>-burler-fix`.
A standalone `lyx burler run` builds its profile with no prefix and keeps its roles.

Tests:
- `TestBouncerEntry_EnvJudgeFallback`'s `callAndCaptureSpec` takes extra config keys, and two rows set `role_prefix: plan` and want `plan-bouncer-seed` and `plan-bouncer-judge`; the existing rows keep the unprefixed roles.
  Each row also asserts the fake shuttle's `GotAttachSpec`, the spec of the Attach that `runSeedSpawn` (seed path) or `judgeCall` (judge path) runs on its own spawn spec before spawning, carries the same role as the spawn.
  The test's doc comment names the role it now pins beside the env fallback.
- `TestBouncer_ReBounceProbesForALiveSeed` and `TestBouncer_EntryProbe_SpecNamesTheJudgesOwnOutputFiles`, the tests that reach the live probes, each gain a row or case that sets `RolePrefix: "plan"` on the fixture's public `Config` before `Build` and asserts `GotAttachSpec.Role` is `plan-bouncer-seed` (the re-bounce probe, `awaitLiveSeed`) or `plan-bouncer-judge` (the entry-time judge probe, `awaitLiveJudge`).
- `TestBouncerEntry_ConstructionFailures` gains a subtest: `role_prefix: Plan!` is refused, the error naming `role_prefix` and the value.
- `TestBurlerHalfEntries_ColorSegment` gains, per entry, a subtest with `role_prefix: plan` that captures `profile.RolePrefix == "plan"`, an absent-key subtest capturing "", and one refusing `role_prefix: Plan!`; its doc comment says it pins both segment keys.
- `TestEngine_Review_TableShape` gains a `rolePrefix` column and a row with `merge`, whose one started seat is `merge-burler-review`; `fakeShuttle.scriptFor` picks the review script for any role ending in `reviewHalfRole`, so a prefixed reviewer still gets it.
  The fake's `started`, `specs`, `gates` and `live` maps stay keyed by the role the seat ran under, so the prefixed row reads its spec and gate under `merge-burler-review`, and the unprefixed rows under `reviewHalfRole` as today.
- `TestEngine_Fix_TableShapeAndFiles` sets `p.RolePrefix = "merge"` and wants the one started seat `merge-burler-fix`, reading its spec and gate under that role.

`shedrecipe`'s `doc.go` lists `role_prefix` among the three rows' keys, `shedadapters`'s `doc.go` Bouncer roles passage says a configured prefix leads the seed and judge roles, and burlerengine's seat-table passage, which names the strands `<shortname>:<slug>:burler-review` and `<shortname>:<slug>:burler-fix`, adds that a profile's `RolePrefix` leads both roles.

**ImpactSummary:** one optional config key on three registry rows; with the key absent every role is today's, so no shipped recipe changes behavior until card 14 sets it.

**Commit:** 13: shedrecipe: optional role_prefix on the Bouncer and Burler rows
