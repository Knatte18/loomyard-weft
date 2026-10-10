# Publish verify log is overwritten by a later run, so the failure is lost

`verify-log-overwritten` · Note · bug

Imported from [issue #502](https://github.com/Knatte18/loomyard/issues/502).

Seen (publish-recovery, 2026-10-10): the Publish verify failed once (exit 1, the #497 subtest re-run), and a later verify run overwrote `.lyx/verify/verify.log`, so Friction-Reflect could not read the original failure and reflected on the later, passing log.

Workaround: none; the failure was reconstructed from the drive report and the orch's own investigation.

Expected: the verify log is kept per attempt (for example `verify-<attempt>.log`, or appended with an attempt header), so a later run never erases the evidence of an earlier failure.
