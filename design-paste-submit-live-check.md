# No live test that a paste followed by Enter submits in Claude Code

`paste-submit-live-check` · Note · enhancement, shuttle

Seen at pane-inject-safety PR-Gate (PR #548): the paste-then-Enter submit rests on each provider's `PasteSubmits()` answer, and no live test checks that Claude Code takes a bracketed paste followed by Enter as a submit.

Idea: a tmux-tier or llm-tier test that pastes a short prompt into a live session and checks it was submitted, so a Claude Code change that breaks it fails a test instead of a run.