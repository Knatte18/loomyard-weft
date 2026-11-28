# Forking the KWin probe batch and its operator steps

Batch 01 held cards 1–4 behind an operator step that sessions cannot take, so it was resumed with a new fork about a dozen times,
once for each change of screen setup, each rewrite of the probe script and each relayed result.
The probe assumptions (one external screen, `eDP-1` alone, hands off the test windows) were each settled only after a run had already failed or been blocked,
and recordings twice had to be thrown away or kept uncommitted because window captions held an e-mail address.

The batch 11 fork also noted, from reading `internal/kwinplace/places.go`, that a window whose home desktop no longer exists is placed elsewhere but keeps the old home.
It left this unchanged as outside its cards, and nobody has verified it.
