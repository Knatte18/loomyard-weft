# Plan round 2 fix: card 1's push-retry ordering test (F2)

The review's proposed fixture for a deterministic remote `cannot lock ref`, a branch `x/y` pushed to a remote holding `x`, does not produce that reason with a local bare remote: git reports `(refname conflict)`, and a held `.lock` file reports `(reference already exists)`.
Both fall outside card 1's retryable forms, so a fixer who applied the fix as written would have planned a test that can never reach the path it pins.
Finding a fixture that works took several manual probes, ending on a `proc-receive` hook, which can answer any reason.
A reviewer proposing a git fixture by its refusal text should probe it first, or say that it is untested.
