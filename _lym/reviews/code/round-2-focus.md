---
round: 2
focus:
  - "internal/hibernateengine/verbs.go Submit: with the switch off, a done or failed request keeps its hub pause entry live and recorded in `Owned`. Settle whether a second `lym hibernate request` over such a request can lose that reference while the entry stays live, and whether `lym hibernate cancel` can still lift it."
  - "internal/hibernateengine/watch.go Watch and its keep path: when `recordOwned` fails and the entry is kept live, settle whether the request written done records that entry, so `lym hibernate cancel` can lift it."
  - "internal/hibernateengine/watch_test.go TestWatch: settle whether rows pin a failure before the hibernate lifting the entry for a request with `ResumeOnWake` false, the failure of the resuming-phase write keeping the entry, and the notice for a request that owns no pause."
  - "internal/hibernatecli/cli.go chooseResumeOnWake: settle whether a `hibernate.yaml` that fails to load refuses the request, and whether the flag given still wins without reading the file, each pinned by a test row."
---

Focus targets the switch's failure and keep paths, where the first review found gaps.
