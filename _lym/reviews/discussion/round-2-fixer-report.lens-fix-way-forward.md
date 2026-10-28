```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: check that every way forward the fix introduced can be carried out from the state it names.

Ways forward the fix introduced:
1. D3's new LLM-driver sentence: the driver stencil skips a re-launch on `binary_changed` and starts no binary watch for a blocked stop "whose reason opens with the escalate prefix". The stop then goes to the arbiter first, and only the arbiter's resume or the orch's `lym loom resume` moves it.
2. D5's driver stencil and guide bullet, which states the same skip.
3. D5's status-spec bullet.
4. The rewritten driver acceptance criterion, which covers the re-check, open-attempt and fresh-start paths.
5. The new acceptance criterion for the LLM driver stencil.

Findings:

- MEDIUM. Changed text: D3's LLM-driver sentence and D5's driver stencil bullet ("skip a stop whose reason opens with the escalate prefix").
  Summary: the LLM driver can only carry out this skip by matching the text of the reason. Its stencil forbids branching on text, and its binary-watch exit path cannot see the reason at all.
  Evidence:
  - `contracts/stencils/shed/shed-template-driver.md:25`: "every branch below is on an envelope field, a policy word or one of the six error kinds, never on a row or recipe name".
  - Line 72: "read that key and never match error text".
  - The escalate prefix belongs to `landingshed` and is specific to loom and darn. The template is shared by all recipes.
  - `contracts/stencils/shed/shed-template-driver-guide.md:67`: the watch's exit re-steps on `binary_changed`, the clean-tree guard and `hand_back` alone, all read from `lym shed status`. The status exposes no field that marks the escalate.
  - So the step can only be carried out by a prefix match on prose the stencil tells the driver never to match.
  - A way forward that works would be a recipe-neutral boolean that Go emits in the envelope and in the status, the way `hand_back` is emitted (for example, a mark meaning "do not re-step on a binary change"). The stencil would branch on that field, and the D5 status-spec bullet would document it. Otherwise D3 needs to state why this one branch on reason text is allowed.

- LOW. Changed text: the new acceptance criterion ("The LLM driver stencil names the escalate halt among the stops it never re-launches on a binary change and starts no binary watch for").
  Summary: the criterion only pins wording in the stencil, so nothing tests that the driver behaves this way. A Go-emitted field like the one above would make the behaviour testable through `loom status` and the envelope.
  Evidence: the stencil is prose that `internal/loomcli/driverprompt.go` fills in, so text is the only thing a test can check.

Tool calls:
- One grep and read of `shed-template-driver.md` (its reason and error lines) and the guide's binary-watch section. My context held only stencil lines 90–150 and the guide's grep hits, not the stencil's branching rule at lines 25 and 72 or the full watch procedure.
