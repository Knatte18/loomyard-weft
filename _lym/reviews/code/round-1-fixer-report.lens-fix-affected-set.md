```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

`hubStartGuard` (`startguard.go:125`) is the only production wiring of the `startGuard.FilingMarked` seam. The new subtest in `beamland_integration_test.go` drives that wiring: it refuses `g` before the mark and passes it after. The other production reads of `shedrun.FilingMarked` (`commitstatus.go:144`, `cleanup.go:250`, `loomcli/wiring.go:280`) are separate from the seam. Each is tested elsewhere or was outside F1.

Tool calls made:
1. Ran one grep for `FilingMarked` and `startGuard{` across `internal` and `cmd`. My context lacked a listing of every site that wires or reads the seam.
