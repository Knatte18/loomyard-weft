# Batten notices without absolute paths and attach tail

`batten-notice-lean` · Note · enhancement, batten

Seen (2026-10-10): every `[batten notice]` injected into the orch session carries absolute paths (`output: /home/.../<slug>/.lyx/verify/verify.log`, `report: /home/.../<slug>/_lyx/shed/<slug>/drive-reports/...`) and ends with `cd <pair> && lyx reed attach`.
The orch knows the hub and pair layout and composes the attach command itself from its role, so the tail and the repeated hub prefix are tokens paid on every notice, in an Opus session whose every turn re-reads its context.

Idea:
- Paths in a notice are relative to the task pair (`.lyx/verify/verify.log`, `_lyx/shed/<slug>/drive-reports/...`), since the slug already names the pair.
- Drop the trailing `cd <pair> && lyx reed attach` from the notice.
