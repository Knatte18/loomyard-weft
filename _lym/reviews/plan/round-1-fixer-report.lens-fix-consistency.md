```yaml
lens: fix-consistency
outcome: reported
```

Task: check whether the round-1 fix change set agrees with the plan cards, decisions and cross-references around it (fix-consistency lens).

- **LOW**: `_lym/plan/00-overview.md`, the intro (line 12) and the Card Index line for card 7. They were not updated alongside the changed Shared Decision "seats come from the table builders".
  - The intro still says "A scan fails on a launch-spec literal or seat table the test does not list", and the Card Index says "the scan for unlisted launch-spec literals and seat tables".
  - The changed decision and card 7 now also make the scan fail on an unlisted `seatengine.Seat` literal that sets `Stencil`.
  - Evidence: the changed overview decision reads "or a `seatengine.Seat` literal that sets `Stencil`, whose function no entry names", and card 7 adds that bullet to `unlistedLiterals`. The overview's twin lines still name only spec literals and seat tables.

No other disagreements:
- Every reference to `Seats`, `Profile.SeatTables`, the Unmet skip, `TestStandingPassages` and the `Omitted` verdict was updated in cards 0–8.
- `manifestEngine` is created in card 6 and used by card 8.
- `Unmet` keeps its production caller in card 8's `report`.

Tool calls: none.
