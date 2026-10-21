# Support log — reed-small-fixes

## Interview

A darn run holds no interview; the writer read the board entry and the code and made the calls in the decision record itself.

## Rejected alternatives

- Learning `allow-set-title` support from the first failed `set-option` and skipping later ones: a failure for another reason, such as a pane gone mid-launch, would disable the option for the whole process, and the board entry asks for the version the floor check already knows.
- Probing `tmux -V` a second time at launch: the floor cache already holds the line's answer, so a second probe costs a subprocess per path for nothing.
- Capping the watchdog's wait to the stop floor while unwatched: it would poll every second without backoff while idle, against PATTERN-daemon-wakeups.
- Making the watchdog command test skip without inotify: the fallback makes the test pass without a free instance, and a skip would hide a broken stop.

## Review rounds

_No rounds yet._

## Question ledger

- Should the fallback log once per process or once per tmux path? Self-picked: once per resolved path, from the floor cache that is already per path.
- Should `tmuxkit.ServerEnviron`'s literal change too, though the board entry names only test files? Self-picked: yes, it is the same defect in the kit those tests pin.
- Does the watchdog fallback also cover a closed event channel? Self-picked: yes, a closed channel delivers no events either.
- How does `TestWatchdogDaemon` keep its no-stop loop? Self-picked: an opener returning a channel that never delivers.
