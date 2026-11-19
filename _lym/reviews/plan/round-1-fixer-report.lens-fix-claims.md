```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that every sentence the fix added or rewrote holds against the code at HEAD.

no findings

The claims I checked, all of which hold:
- **Card 19:** an `efforts` key is CategoryNone and `skills` is CategoryGuard (`internal/agentconfig/config.go:123-124`). The current refusals send the fix "from the prime" (`config.go:77`).
- **Card 8:** the parent pids come from the same listing, since `proc.Process` carries `PPID` (`internal/proc/process.go:23-24`) and `ListProcesses` returns it.
- **Card 11:** the `harden-commits-&lt;round&gt;.json` signal is written on a verify-exhausted round. That round returns a nil-error counted Stuck (`internal/shedadapters/burlerfix.go:325-332`), which card 9's step 4 counts as "no error". The fixer report is archived away on that path (`burlerfix.go:127-128`), as the fixer judged. The unjudged hand-on Stuck (`burlerfix.go:176`) only reaches a round that already holds a fixer report, so it raises no false red.
- **Cards 6 and 14:** a refusal of `lym gate test` exits 1 with a JSON error envelope on stdout (`internal/gatecli/doc.go:1462`), and a refused tier prints only its JSON envelope (`doc.go:843`). That supports parsing `Refused` from the envelope.
- **Card 2:** `harden-burler-fix` takes `code-burler-fix`'s reports, `[reach, redundancy, mutation]` (`internal/gateslot/template.yaml:40`).
- **Overview:** the `{tiers: [...], scope: diff}` writes keep the template's reports (`internal/gateslot/doc.go:120`).
- **Cards 7 and 16:** `hardenshed` and `hardencli` lie outside the fabric owner set, and `fabricengine` lies inside it (`pattern/PATTERN-fabric-boundary.md:14-15`).
- **Card 8:** the new `PATTERN.md` line is 280 characters, and the cap check refuses only `n &gt; MaxEntryTextChars` (`internal/pattern/check_test.go:89`).

Tool calls made:
- Grep of `agentconfig` categories and `burlerfix.go` early returns: needed to confirm card 19's category claim and card 11's commits-file signal.
- Grep of `gatecli` for the envelope's JSON fields: needed for card 14's envelope parse claim, and it found none in production code.
- Grep of `gatecli/doc.go` for the envelope contract: needed to locate the refusal envelope rule.
- Read of `gatecli/doc.go:1436-1470` and `836-846`: needed to confirm that a refusal prints a JSON error envelope and exits 1.
