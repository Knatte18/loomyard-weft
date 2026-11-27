# Fixing code review round 1 of pr-review-stalls-no-session

F2 asked for the PR-Review tests to run over a shared kit that carries a recorded LLM session stand-in, with reed and creel real.
No kit carries a recording, a real reed needs tmux while these tests spawn nothing, and `shuttlefake` is itself listed as a module stand-in.
Only part of the finding could be done, and the self-check then counted the new stand-ins the partial fix needed.
A review fix that names test infrastructure would help by checking that the infrastructure exists.
