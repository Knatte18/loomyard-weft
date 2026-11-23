```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Set-premise findings I checked:
- **F7 (sites that archive judge outputs for a re-judge):** covered.
  - `rejudge` at `bouncer.go:1220` and the hard-cap halt at `bouncerdecision.go:210` now both archive the verification with the judge outputs.
  - The other archive sites are not re-judge retirements: `retireContinue` archives the escalation record, parent notice and decision file, and the fresh-judge run at `bouncer.go:2191` archives only its own outputs.
  - `verificationPath` has no other reader or writer. The live-verifier path in `bouncerverify.go` archives only its own output.
- **F8 (writers of the ledger):** covered.
  - `markLedgerResolved` was the only non-test writer of the ledger.
  - The grep for `WriteFile(ledgerPath` and related forms found no other site.
- **F1/F12 (consumers of the verification format):** covered.
  - `parseVerification` and `verificationEntryYAML` have no use outside `bouncerverify.go`.
  - The stencil, the one place that tells the verifier what to write, now carries `finding`.
  - The marker set in `markers_test.go` is unchanged.
  - `shed-recipe-spec.md:130` and `docs/overview.md:1523` describe the verifier at a level the change does not alter.
- **F9 and F10:** each has a single site, and both are fixed.

Unchecked: F2–F6 are test rows in one test function, not sets of sites, so I did not rerun a search for them.

Tool calls (3 of 6):
1. Grepped `archiveStaleOutputs(`, `verificationPath(` and the verification file name, ledger writes, and docs naming the verifier format. My context lacked a repo-wide enumeration of those sites.
2. Read `bouncer.go:1355-1375` and `2150-2195`. My context did not show what the other archive sites at `bouncer.go:1366` and `2191` archive.
3. Grepped the recipe spec, `docs/overview.md` and `stencils.go` for verification-format mentions. My context did not show whether those docs describe the verification file's fields.
