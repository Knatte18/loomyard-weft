# Batten notices with their own text no longer name the child state

`notice-custom-state` · Note · enhancement, batten

Seen at batten-notice-trim PR-Gate (PR #551): the notices with their own text (quiet, api-error, driver-dead) lost the `child state X` clause when the condition was rephrased, so they no longer say which state the child is in.

Idea: carry the state in the rephrased condition for these notices too, as `haltText` does for halts.