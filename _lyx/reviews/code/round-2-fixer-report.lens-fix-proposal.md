```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposal for F1, which moves `restore-plan` from the operator audience to the role audience.

**Finding (LOW)**
- **Changed text:** `internal/webstercli/restoreplan.go:19`, `Annotations` set to `clihelp.AudienceRole`.
- **Summary:** The command index lists a command under one audience only, so `restore-plan` drops out of the operator index although operator-facing text still tells the operator to run it.
- **Evidence:**
  - `internal/clihelp/index.go:54` includes a command only when its annotation equals the rendered audience, and `AudienceOperator` is the default for `--audience` (`internal/helpcli/cli.go:103`).
  - The operator is still told to run it:
    - `contracts/stencils/webster/webster-template-master.md:252`: "the operator ... runs ... `lyx webster restore-plan`".
    - `internal/webstercli/validate.go:160` and `internal/webstercli/run.go:61` name it in the refusal text.
    - `internal/websterengine/findingtext.go:36` and `internal/websterengine/beginbatch.go:65` name it as the way forward.
  - The sibling `rebaseline` was already role-audience with the same dual use, so the change matches existing practice and is not a regression.
  - The reviewer's own check ("no test or generated index pins restore-plan under operator") holds:
    - My grep of `clitree_test.go`, `helpcli/cli_test.go` and `docs` found no pin.
    - The parent's earlier `lyx gate test` run over `./internal/webstercli`, `./cmd/lyx` and `./internal/helpcli` passed.
  - No fix is needed. The operator can still run it by name, and the refusal messages surface it.

No other case where the proposal is wrong. A command carries one audience annotation, so the role index is the only listing the plan fixer's prompt needs.

**Tool calls**
1. `grep -n "Audience" internal/helpcli/cli.go internal/clihelp/*.go`: confirmed that the index filters by a single audience, which my context lacked.
