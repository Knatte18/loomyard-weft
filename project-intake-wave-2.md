# P3 Intake wave 2: Describe text, infra stops, push retries

Rendered 2026-11-26 09:45 UTC; every age below is as of then.
Under: [Board](README.md).

`intake-wave-2` · Age 1h · Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Draft (3)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T9 | `draft` **Review the Describe text in its row, keep LoomYard terms out, sync the PR body** | [`describe-review-and-sync`](design-describe-review-and-sync.md) | darn · solid | T3<br>T1 | 2h<br>1h | landing, stencil, burler, shuttle |
| T10 | `draft` **Machine stops, dead judges and idle seats are not producer failures** | [`infra-stops-not-failures`](design-infra-stops-not-failures.md) | darn · solid | T3<br>T4<br>T5 | 2h<br>1h | shuttle, claude, tender, gate, burler, shed, reed |
| T11 | `draft` **Background pushes retry quietly; landing pushes and reads retry instead of failing** | [`push-retry-not-halt`](design-push-retry-not-halt.md) | darn · solid | T3<br>T1<br>T9<br>T10 | 2h<br>27m | fabric, shed, landing, batten, dobby |
