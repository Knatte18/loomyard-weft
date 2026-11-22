# Card 2 — finalize-notice-skips-autonomy

**Edit:**
- `internal/landingshed#Finalize.configChangeNotice`
- `internal/landingshed#TestFinalize_ConfigChangeNotice`

**Uses:**
- `internal/beamsetting#Module`
- `plan:internal/beamsetting#AutonomyKey`

**Intent:** D1: a worktree's autonomy is its own, so Finalize's config-change notice never asks an orch to re-apply a child's `autonomy` on its parent.

`configChangeNotice` skips a changed key whose module is `beamsetting.Module` and whose key is `beamsetting.AutonomyKey`.
A file whose only changed key is `beam.autonomy` names nothing, and a file that also changes another key names only that other key.
Its comment says which key it leaves out and why.

`TestFinalize_ConfigChangeNotice` (tier0) gains two rows over a `beam.yaml` change:
one changing only `autonomy` from `false` to `true`, whose notice is `""`,
and one changing `autonomy` and `max_runs`, whose notice names `beam.max_runs` and does not contain `autonomy`.

**ImpactSummary:** One skip inside the notice's key loop; every other module and key is named as today, and the notice's one caller in Finalize is unchanged.
