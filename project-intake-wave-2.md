# P3 Intake wave 2: Describe text, infra stops, push retries

Rendered 2026-11-27 10:18 UTC; every age below is as of then.
Under: [Board](README.md).

`intake-wave-2` · Age 1d · Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Layer C (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T11 | **Background pushes retry quietly; landing pushes and reads retry instead of failing** | [`push-retry-not-halt`](design-push-retry-not-halt.md) | tapestry | T6<br>T8 | 1d<br>3m | fabric, shed, landing, batten, dobby |
