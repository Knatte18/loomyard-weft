```yaml
lens: fix-affected-set
outcome: reported
```

Task: check, through the fix-affected-set lens, that F2's exit-reason change reaches every site that describes it.

No findings.

The one doc site the grep turned up that restates the failed-end behaviour is `internal/orchcli/doc.go:120`. It says the strand is removed even when the end failed. That stays true, and it makes no claim about the exit reason, so the F2 change doesn't contradict it. The other sites that describe this behaviour are `owner.go` and `orchengine/doc.go`, and the fixer updated both.

F1 is left unchecked: it targets `_lym/landing/summary.md`, which is outside the fixer's write surface. F3 is also left unchecked: it adds one section to a package doc and names no set.

Tool calls:
1. Bash grep of `internal` for the wording around failed ends and the exit reason: my context lacked any search for other docs or code describing it.
