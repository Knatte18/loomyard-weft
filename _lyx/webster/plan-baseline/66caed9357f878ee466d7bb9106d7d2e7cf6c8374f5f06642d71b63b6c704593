# Card 5 — no-denied-recovery-forms

**Edit:**
- `cmd/lyx#deniedRecoveryForms`
- `cmd/lyx#TestNoDeniedRecovery_ScanSelfCheck`
- `pattern/PATTERN-no-denied-recovery.md`

**Intent:**
D7: `PATTERN-no-denied-recovery`'s closed list follows `claudeengine`'s deny table, which card 3 grew by the in-place deny.

`deniedRecoveryForms` gains `{"sed", "-i"}` and `{"perl", "-i"}`.
These catch only the bare spellings; the other spellings card 3 denies (`-i<suffix>`, `--in-place` and its abbreviations such as `--in` or `--in-pl=.bak`, and a short cluster holding `i` such as `-ni` or `-pi`) stay outside the token scan, as the raw `go` deny does.
The computed-path `rm` deny of card 4 is pattern-shaped and stays outside the list too, so a plain `rm <path>` stays a valid step.
The scanned surface holds no `sed -i` or `perl -i` today, and card 1's directive names each after its own "never", so `TestNoDeniedRecovery_ScannedSurfaceHasNone` stays green.

`TestNoDeniedRecovery_ScanSelfCheck` gains rows: "way forward: sed -i s/a/b/ f" and "way forward: perl -i -pe s/a/b/ f" each give one hit;
the directive's own wording "never `sed -i`, never `perl -i`, never `awk`" gives none;
"read it with sed -n 3p f" gives none;
and "way forward: sed -i.bak s/a/b/ f" gives none, pinning that a suffixed spelling is outside the scan.

`pattern/PATTERN-no-denied-recovery.md`'s closed-list sentence adds `sed -i` and `perl -i`.
One new sentence, beside the raw `go` paragraph, says the computed-path `rm` deny and the in-place spellings beyond the bare `sed -i` and `perl -i` are outside the scan.
The `PATTERN.md` entry line is unchanged.

**ImpactSummary:** Widens one test-only closed list and its self-check, and the pattern background file that describes it; no production code changes.
