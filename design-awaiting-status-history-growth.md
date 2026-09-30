Batten polls a child in `awaiting` every 30 s and each poll appends to `status.json` history.
The wait is unbounded by design, so collapse repeated identical wait entries or cap them.
Related: right after a resume a poll can still see `awaiting` and logs a misleading re-approve hint (`internal/battenshed/innerrun.go`).
Found reviewing PR #304.