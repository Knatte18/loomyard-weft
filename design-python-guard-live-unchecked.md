# The Python guard's new refusal text was never seen in a real session

`python-guard-live-unchecked` · Note · research, shuttle, claude

## Seen

The landing of `python-guard-gives-no-reason` (`b002ca9a`) states in the refusal and in the appended system prompt why Python is refused and how to lift it.
Only tests and the settings golden file cover the wording; no real lym-spawned session was run to see the refusal and the system prompt read as intended.

## Idea

In the sandbox hub `lyt`, open one lym-spawned session, have it try `python3 --version`, and record the refusal it gets and the system prompt line it carries.
