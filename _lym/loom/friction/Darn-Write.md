# Running the gate for the VS Code colour change

Two `lym gate test --wait` runs and one `go vet` failed with `fork/exec ... resource temporarily unavailable` and `runtime: failed to create new OS thread (errno=11)`, even though `ulimit -u` was 113180 and only about 3200 threads were live.
Builds and test binaries failed in packages this change never touches, and a PreToolUse hook also failed once with "Cannot fork".
A third run a minute later was green, so the limit seems to come from a short-lived pressure, probably the other runs in flight sharing the machine, not from the change.
A gate that reports these as test failures, instead of as a retryable environment error, makes the cause harder to see.
