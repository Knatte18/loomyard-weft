# Batten notices drop the pair path and attach command

`batten-notice-no-attach` · Note · enhancement, batten

Every `[batten notice]` ends with the pair's path and its attach command, such as `; cd /home/knatte/Code/loomyard-LYXHUB/bugfix-fabric && lyx reed attach` (`internal/battenshed/notice.go`).
The notice goes to the orch, not to the operator: the orch acts on the slug and state, and when it reports to the operator it gives the attach command itself.
In the orch's pane the long path and command are clutter on every notice, and they crowd out the part that matters, which is sometimes cut short (seen 2026-10-07: a circling notice ended in "decide with `lyx loom c; cd …").
Wanted (operator, 2026-10-07): the notice carries the slug, the state and the error, and drops the path and attach command.
