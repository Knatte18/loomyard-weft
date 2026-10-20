# Clock the unit and integration tiers on OSL-1046 with Cortex XDR

`bench-osl1046-xdr` · Note · audit

## Seen (2026-10-20)

On the work laptop OSL-1046 (AD-joined Ubuntu 24.04, Cortex XDR running), the darn verify of `arbiter-verdict-quoting` (`lyx gate test ./... && lyx gate test --tags integration ./...`) took 7.5 min (08:07:00 to 08:14:31 local), with the run's own agents active beside it.
On the operator's home machine Neve the integration tier alone takes 56 s (`docs/benchmarks/test-suite-weight.md`).

## Operator order (2026-10-20)

Once `arbiter-verdict-quoting` has landed and the machine is quiet, clock the unit tier and the integration tier on this machine separately, and log them in `docs/benchmarks/` beside the other timings as their own "OSL-1046 with Cortex XDR" numbers, comparable with Neve's.
Record as much hardware information as possible (CPU model, cores and threads, RAM, disk type, kernel, Go version, the gate's `go_parallel` and slots) for both machines where the doc lacks it.
