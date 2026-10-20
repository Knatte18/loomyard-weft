# Three tests spell LYX_TRACE_ID as a literal instead of logger.TraceIDEnv

`trace-id-env-literal` · Note · enhancement, reed, shuttle

## Seen (Darn-Review of `gate-fewer-new-binaries`, 2026-10-20)

The run made its sites name the trace id variable through `logger.TraceIDEnv`, and the reviewer deferred three test files outside the finding's scope that still spell it `"LYX_TRACE_ID"`: `internal/reedengine/lifecycle_test.go`, `internal/shuttleengines/shuttleengines_test.go` and `internal/testkit/tmuxkit/tmuxkit_integration_test.go`.

## Idea

Those tests use `logger.TraceIDEnv`, so a rename of the variable cannot leave them testing a name lyx no longer sets. No effect on run time.