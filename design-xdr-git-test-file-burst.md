# Git test fixtures trip Cortex XDR on the work laptop

`xdr-git-test-file-burst` · Note · enhancement, fabric, deploy

## Seen (operator report, 2026-10-16)

On the operator work laptop, which runs Cortex XDR, the git tests create so many files at once that XDR flagged them as malware behaviour.
Twice it terminated all of VS Code, and both times XDR itself crashed afterwards, so the machine had to be restarted.

For scale, on the home machine a parallel `lyx gate test --tags integration ./...` (gate `go_parallel: 4`) takes 56s, and the serial package-by-package total is 156s (`docs/benchmarks/test-suite-weight.md`, which also counts git fixture and code subprocesses per package).

## Idea (orch recommendation, not a decision)

- Short term: on that machine set the machine-local gate layer, `lyx config gate --layer local --set go_parallel=1` and `--set slots=1`, so one package and one test run go at a time.
- Check on the laptop whether that is enough, since XDR may react to the file-creation rate within one package as well.
- If not: make the git fixtures create fewer files (shared fixtures built once per package, cloned or copied less), and measure files created per package so the heaviest are known.
## Seen (orch, 2026-10-20): XDR holds every new binary on its first runs

On OSL-1046 the full verify took 7.5 min with the CPU mostly idle.
Measured with the machine quiet:

- 200 runs of `/bin/true` take 0.23 s, 100 runs of `git --version` 0.27 s, and 1000 small file writes 0.04 s, so known binaries and file creation are not slow.
- A freshly built ten-line Go binary took 10.2 s on its first run and 7.8 s on its second, with about 0.01 s of CPU each; a copy of it made after that ran in 0.04 s.

So Cortex XDR holds each executable it has not seen yet for several seconds before it runs, with no CPU used, while it analyses it (its `sandboxd -n Yara` and `pmd` processes run on the machine).
`go test` links a new test binary for every package on every run that changed it, and lyx's own tests build and run further binaries, so each package pays this wait.

## Idea

- Ask IT for an XDR exclusion of the Go build cache and the temp dir `go test` links its binaries into (`go env GOCACHE`, `GOTMPDIR`), or of the operator's code tree, and measure again.
- Without an exclusion, point `GOTMPDIR` at one fixed directory so an exclusion can be narrow.
