# batten cancel --remove of a beam leaves the project's beam flag set

`beam-cancel-keeps-flag` · Note · bug, batten

Imported from [issue #259](https://github.com/Knatte18/loomyard/issues/259).

A beam was made with lym beam create <slug> from the prime, then ended before any member ran with lym batten cancel <slug> --remove. The cancel ended the pawl run and removed the pair and its records, but the board project kept "beam": true and had status draft.

The flag cannot be cleared afterwards. lym board upsert '{"slug":"<slug>","beam":false}' is refused: 'the beam flag stays set for the beam's life, which ends when lym beam land <slug> lands it'. But the beam has no pair and no run left to land. The only way back to a plain project was to detach the members, remove the project, upsert it again and re-attach the members and their depends_on, which changes the project's handle.

The cancel also warned: 'the beam's orch was not stopped: lym orch runs from the hub's prime worktree or a beam; "<slug>" is neither', which shows the cancel no longer saw the entry as a beam once the pair was gone.

Expected: batten cancel --remove of a beam's pawl run ends the beam's life and clears the project's beam flag, leaving a plain project whose members are untouched. Alternatively, a verb such as lym beam cancel <slug> does so.