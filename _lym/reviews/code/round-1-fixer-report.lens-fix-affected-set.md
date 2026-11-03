```yaml
lens: fix-affected-set
outcome: reported
```

Every acquirer of `fabric.push.lock` writes a holder record: `pull.go:500`, `pendingflags.go:139` and `originreconcilemoves.go:96`. `pushanchored.go:61` only reports.

No findings.

- Ran one grep for `holdPushLock(` callers outside tests. My context lacked a search of every caller, and F5's premise is a set of sites.
- Ran one grep for `weftPushLockFile` outside tests. It confirms no other acquirer of the lock skips the record.

I did not check F2 (left unchanged, no context in `PushTask`), and F1 is out of scope as a decision finding.
