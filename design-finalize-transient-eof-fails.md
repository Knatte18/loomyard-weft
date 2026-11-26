# Finalize halts as hard failure on a GitHub API EOF that lym marks transient

`finalize-transient-eof-fails` · Note · bug, landing, shed

Imported from [issue #240](https://github.com/Knatte18/loomyard/issues/240).

A `lym loom step` at the Finalize producer read a pull request's status from the GitHub API and got `unexpected EOF`.
The step was refused with `transient=github-api`, yet the run halted as `failed` with anomaly `producer-hard-failure`.

Quoted from the trace:

```
level=WARN msg="shed: step refused" kind=producer transient=github-api error="landingshed: Finalize: read the pull request's status: pullrequest: read pull request #54: Get \"https://api.github.com/repos/<owner>/<repo>/pulls/54\": unexpected EOF"
```

Halt note text: `loom halted failed at Finalize`, `state: failed`, `anomaly: producer-hard-failure`, 15 history entries, all earlier steps done or skipped.

This is abnormal because lym itself labels the error transient, and a single dropped connection to the API then ends a run that had finished every earlier step.
Other network failures in the same run (an unreachable origin before a push) were retried until they landed, so the Finalize read is inconsistent with them.

Suggested fix: when a step is refused with a non-empty `transient` kind, retry it with backoff, or leave the run resumable, instead of recording `producer-hard-failure`.
A failure of this kind should reach `failed` only after the retry budget is spent.

Provenance: task `transient-tripout-overpull`, from `loom-halt-2.md`.
