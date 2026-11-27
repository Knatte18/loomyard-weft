# Coordinating implementer forks: foreground gate calls moved to the background

Forks reported, in most batches, that the harness moved a `lym gate` call to the background after 180s although the brief says to run it in the foreground.
Several of those first calls ended with the slot-busy exit 75 and had to be run again.
Each wait reached Master as a "fork is waiting on a command" notification, which is not a completion, so Master took many turns that did nothing.
No batch failed or needed recovery.
