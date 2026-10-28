```yaml
lens: fix-claims
outcome: reported
```

Task: check the claims in the fixer's added or rewritten text against the code at HEAD (fix-claims lens).

Claims checked:
- D2 (`decision-record.md:49`), "it leaves the only procedure fan that exempted it": **holds.** `way-forward` sits in `review-plan` and `twill-plan`, and in no other `review-*` or `fix-*` fan (`internal/lenses/template.yaml:403,414`).
- D2 (`decision-record.md:49`), "as the other `twill-plan` lenses end": **does not hold** (finding 1).
- D5 (`decision-record.md:80`), Migrate runs in `PrepareRunSettings` and in a reconcile of the local file: **holds** (`internal/configreg/runsettings.go:73-84`, `internal/configsync/configsync.go:562-563`).
- D5 (`decision-record.md:80`), a hand-edited local file is refused at load: **holds.** `webster_reviewer_fan` is not in the loom template and not in `RetiredKeys`, so `ValidateLocal` refuses it with "not in the template" (`internal/configengine/layers.go:283-284`).
- D5 (`decision-record.md:80`), "as for every retired key": **does not hold** (finding 2).
- D5 (`decision-record.md:85`), "point it at any other fan `lenses.yaml` defines": **holds.** `ValidateFanKeys` only resolves the name (`internal/lenses/fankeys.go:65-78`).
- D5 (`decision-record.md:85`), "keeps the reviewer's whole-diff read under the code-review rubric": I did not verify this, to stay within the call budget. It repeats the unchanged second Bound bullet ("its rubric and the reviewer's read of the whole diff are unchanged").
- Auto-mode assumptions (`decision-record.md:120`), the D1 `review-discussion` line: **holds.** It matches the D1 acceptance line and support-log Q3.
- D2 acceptance line (`decision-record.md:136`): a criterion, not a claim about current code. It is consistent with line 49.

Findings:

1. MEDIUM. D2, `decision-record.md:49`: "Its prose gains the emphasis line after its list, as the other `twill-plan` lenses end".
   Summary: not every other `twill-plan` lens ends with the emphasis line. `guard-reach` ends in its list and stays exempt through `review-plan`.
   Evidence: `twill-plan` is `coverage`, `row-validity`, `forced-edits`, `guard-reach`, `way-forward` (`template.yaml:398-403`). `guard-reach`'s prose ends "Your list: per guard, reachable or not." with no emphasis line (`template.yaml:280-287`). D2 keeps `guard-reach` in `review-plan`. A correct wording would be "as `coverage`, `row-validity` and `forced-edits` end".

2. MEDIUM. D5, `decision-record.md:80`: "a hand-edited local file holding one is refused at load until a reconcile migrates it, as for every retired key".
   Summary: not every retired key is refused in a local file. A declared retired key of category none is admitted. The webster keys are refused only because they are not in the template and not declared retired.
   Evidence: in `ValidateLocal`, a path covered by `retiredCovering` with `CategoryNone` hits `continue` and is admitted (`internal/configengine/layers.go:275-280`). Loom declares `RetiredKeys: {"fan_review": none}` (`internal/loomengine/config.go:555-557`), so a local `fan_review` passes the load check. `webster_reviewer_fan` and `webster_fan` appear only in the rename tables (`config.go:1022,1035`), so they fail with "key ... is not in the loom template" (`layers.go:283-284`). The clause should drop "as for every retired key" or say "as for any key outside the template".

Tool calls:
- `grep` of `template.yaml` for `way-forward`, of the configsync and configengine Migrate sites and of loom's retired declarations: the context lacked the full fan membership and the reconcile-route code.
- `sed` of `template.yaml:395-425` plus a `grep` for `RetiredKeys`: the context lacked the full `twill-plan` list and where loom declares its retired keys.
- `sed` of `config.go:553-575` and `template.yaml:280-288`: the context lacked loom's `RetiredKeys` contents and `guard-reach`'s closing prose.
