# Finalize records a DNS failure classed transient as a failed / producer-hard-failure halt

`transient-stop-not-failed` · Note · bug, shed, loom

Imported from [issue #269](https://github.com/Knatte18/loomyard/issues/269).

## What happened

A `lym loom step` at Finalize failed on a DNS lookup of `api.github.com` while reading the parent branch's rules.
The step was refused with `transient=github-api`, yet the run was recorded as `failed` with anomaly `producer-hard-failure`.

Trace excerpt:

```
22:40:16 level=INFO msg="shed: step"
22:40:18 level=INFO msg="loomcli: no live driver strand to carry the verify wait mark" label="parent lock main"
07:21:41 level=INFO msg="loomcli: no live driver strand to carry the verify wait mark" label=""   (next day)
07:21:41 level=WARN msg="shed: step refused" kind=producer transient=github-api error="landingshed: Finalize: read GitHub's rules for the parent branch: pullrequest: read the rules active on branch main: Get \"https://api.github.com/repos/<owner>/<repo>/rules/branches/main?per_page=100\": dial tcp: lookup api.github.com on 127.0.0.53:53: server misbehaving"
```

Halt note:

```
loom halted failed at Finalize
producer: Finalize
state: failed
anomaly: producer-hard-failure
history_entries: 14
```

A second `lym loom step` started about 0.1 s after the refusal and got through Finalize.
Its post-merge gate passed and the parent branch was pushed.

## Why it is abnormal

lym itself classes the error as transient, but the persisted state and the halt note class it as a hard producer failure.
The halt note then reaches reflection as a defect signal for an outage that cleared at once.

This is the same shape as #240 (a GitHub API EOF at Finalize), seen here on a DNS failure at the rules read.

The step also spent about nine hours between the parent-lock wait mark and the refusal.
The trace does not show whether the lock was held that long or the host was suspended.

## What to do

- Record a failure with a non-empty transient class as a retryable stop, not as `failed` / `producer-hard-failure`, at least until the driver's re-steps are spent.
- Write no `loom-halt` note, or one with its own anomaly kind, for such a stop.
- Log the parent-lock wait's start, end and holder in the trace, so a long wait can be explained.

Provenance: task `reach-misses-tier1-calls`; note `loom-halt.md`.
