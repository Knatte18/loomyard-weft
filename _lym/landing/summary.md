# Name a hub's desktop launcher "lym: <shortname>"

A hub's desktop launcher was titled "lym: start the hub <shortname>", such as "lym: start the hub odd".
That is too long for a desktop, where the part that matters is which repository it is.
It is now "lym: <shortname>", such as "lym: odd".

## Design decisions

- The new title is the `Name=` key of a `.desktop` launcher and the `rem` line of a `.cmd` launcher.
- A `.cmd` launcher shows its file name, `lym-hub-<shortname>.cmd`, which is unchanged.
  The visible name on Windows and Windows with WSL therefore stays as it was.
- The all-hubs launcher keeps its title, "lym: start every hub".
- File names, the command a launcher runs and the `.desktop` `Comment=` key are unchanged.
- A launcher already on a desktop keeps its old title until `Add` writes it again; this change adds no migration.
- The package doc of `internal/desktoplauncher` now says what each launcher form displays.

A later lym icon on the launcher may allow the shortname alone as the name; this change does not do that.

## To check by hand

- On a freedesktop desktop, write a hub's launcher again and confirm it is shown as "lym: <shortname>".
  The tests compare the launcher file's text, not what a desktop displays.

## Merge danger

**Door:** two-way
**Blast radius:** desktoplauncher
