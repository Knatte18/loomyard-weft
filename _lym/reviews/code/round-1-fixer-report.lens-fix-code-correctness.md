```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's change set for code correctness (a changed function, caller or test).

**no findings**

The change set is documentation and stencil text only: four edits to `contracts/specs/loom-plan-checks-spec.md`, one to the plan-review rubric and one to `contracts/stencils/webster/webster-template-plan-fix.md`. It has no function, signature or test change, so there is nothing to run step by step and no caller to break.

- **F4 marker:** the new sentence in `webster-template-plan-fix.md` uses `{{.specs_dir}}`, which the stencil already declares as a required marker. It adds no new marker, and the path form matches the two sentences above it.
- **F4 target file:** `loom-plan-csharp-spec.md` exists in `contracts/specs`. I confirmed that with `ls`, which did not show whether the file sits at the deployed `{{.specs_dir}}/loom/` path.
- **F1 counterpart rule:** the added clause matches the review's description of `planparser.GoPackageCounterpart`, called at `internal/loomshed/seamtest.go:137`. I did not re-read that code.

Tool calls (1):
1. `ls contracts/specs/loom/`, `ls contracts/specs | grep -i csharp` and `grep specs_dir` on the stencil. I needed to know whether the C# spec exists and the marker is declared, and my context did not show either.
