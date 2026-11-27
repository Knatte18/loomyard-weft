---
verdict: CONTINUE
rationale: "Round 1 carries two gating design findings at MEDIUM: F1 (a heredoc body line holding a bare word ends the skipped body, so non-editing commands are denied) and F2 (a shell -c script after a closed heredoc is not read); no finding relabelled."
---

F1 and F2 are both class `design` at severity MEDIUM, so the decision rule gives `CONTINUE`.
F3 and F4 are LOW `design`, and F5 and F6 are LOW `consistency`.
No finding is BLOCKING, none cites a PATTERN entry, and no class or severity is relabelled.

Focus departure, ratified: the reviewer did not read the testdata diff pieces whole,
and instead extracted the rendered shell-edit hook command at HEAD and at the base and ran and measured both.
The review states that the golden is the renderer's output,
and running the rendered hook answers the question the focus entry asked about behaviour and size.
