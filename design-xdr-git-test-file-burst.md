# Git test fixtures trip Cortex XDR on the work laptop

`xdr-git-test-file-burst` · Note · enhancement, fabric

## Seen (operator report, 2026-10-16)

On the operator work laptop, which runs Cortex XDR, the git tests create so many files at once that XDR flagged them as malware behaviour.
Twice it terminated all of VS Code, and both times XDR itself crashed afterwards, so the machine had to be restarted.

For scale, on the home machine a parallel `lyx gate test --tags integration ./...` (gate `go_parallel: 4`) takes 56s, and the serial package-by-package total is 156s (`docs/benchmarks/test-suite-weight.md`, which also counts git fixture and code subprocesses per package).

## Idea (orch recommendation, not a decision)

- Short term: on that machine set the machine-local gate layer, `lyx config gate --layer local --set go_parallel=1` and `--set slots=1`, so one package and one test run go at a time.
- Check on the laptop whether that is enough, since XDR may react to the file-creation rate within one package as well.
- If not: make the git fixtures create fewer files (shared fixtures built once per package, cloned or copied less), and measure files created per package so the heaviest are known.