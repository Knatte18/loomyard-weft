---
round: 2
focus:
  - "internal/weaverengine/doc.go, config.go and template.yaml: is every config key named in a comment or doc written in full as <module>.<dotted path>, as PATTERN-config-key-names asks?"
  - "internal/weaverengine/run_test.go and run_fixture_test.go: do the verify span tests pin each span's Start and End against independent instants, so that swapped or mistimed intervals would fail?"
  - "internal/weaverengine/run.go, finishRun and runFixer: is a rework span recorded only for a verify fixer that spawned, running from its spawn to its end, and does the VerifySpans doc match?"
  - "internal/weaverengine/resume_test.go: is it pinned that a verify fixer the next call attaches to adds no rework span, while the verify runs that follow still record gate spans?"
  - "The AttemptTimeoutMin comment in internal/weaverengine/config.go and the attempt_timeout_min comment in template.yaml: do they agree with how the bound is counted on spawn and on attach to a strand a crashed call left (resumeAttempt, attemptLaunch.Timeout in strand.go)?"
---
