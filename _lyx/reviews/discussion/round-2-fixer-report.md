# Fixer report — discussion round 2

## Changes

All in `_lyx/discussion/decision-record.md`.

- **F1** (fixed as proposed): D2's long form is now any `--` word that is a prefix of `--in-place` at least as long as `--i`, optionally followed by `=<suffix>`, with a sentence saying GNU sed accepts unambiguous abbreviations and no other GNU sed long option starts with `--i`, so the widening denies no read.
  Premise checked against `sed --help`: its long options are `--debug`, `--expression`, `--file`, `--follow-symlinks`, `--help`, `--in-place`, `--line-length`, `--null-data`, `--posix`, `--quiet`, `--regexp-extended`, `--sandbox`, `--separate`, `--silent`, `--unbuffered` and `--version`.
  The in-place deny criterion now refuses `sed --in s/a/b/ f` and `sed --in-pl=.bak s/a/b/ f`.
  D7 lists `--in-place` and its abbreviations among the spellings outside the token scan.
- **F2** (fixed as proposed): D6 now says each further ask folded into a set latch after its notice advances the same persisted dialog offset, so a reattach never rebuilds a latch from an already-notified episode.
  The "persisting the latest one covers them all" sentence now says ask events may land on either side of the send, and a second fork's dialog can fold into a notified episode, and that advancing the offset covers both.
  The first latch bullet's "changes nothing" became "opens no new episode and keeps the latch's stamp", since a further ask now advances the offset.
  The reattach criterion adds the case of a `permission_prompt` `Notification` landing after the notice and before the restart, with no second notice.
- **F3** (fixed as proposed): D6's third latch bullet reads the pane on each poll tick from the moment the latch is set; an input box clears the latch at any age, and a later ask opens a new episode with its own stamp and offset; the notice is still sent only by a no-input-box read once the latch is bound-old.
  The post-notice paragraph is shortened to say the reads go on until an input box shows.
  A new criterion: an ask, an input box at 10 s and a second ask at 55 s send no notice before 60 s after the second ask.

`_lyx/discussion/support-log.md` is unchanged.

## Disputed

None.

## Deferred

None.
