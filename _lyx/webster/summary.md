# Restore SetContentRows

Batch 10 (`10-restore-set-content-rows`) is done at head `533defd3a484ef7c3ef431136dca4eacdc957c2f`.
It restored `Engine.SetContentRows` and the `InstallPins` apply option, with their tests and the `doc.go` paragraph.
`record-batch` returned no deviations and no warnings, so the scope guard found no files changed outside the batch's targets.
