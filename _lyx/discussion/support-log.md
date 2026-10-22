# Support log — csharp-fixtures-compile

## Interview

A darn run holds no interview; the writer read the board entry and the code and made every call itself.

## Rejected alternatives

- A dedicated `dotnet` build tag and gate tier: rejected, since the skip on a missing `dotnet` already keeps a Go-only machine green, and a new tier would need a hub `gate.yaml` change no gate list runs yet.
- Placing the test in `internal/planglyph`, beside the one fixture today: rejected, since the board entry asks for every fixture under `testdata/`, and fixtures may appear in other packages.
- One project for all fixtures: rejected, since fixtures in different directories may declare the same type names.
- One project per `.cs` file: rejected, since files of one fixture directory may reference each other, and each `dotnet build` costs seconds.
- Copying fixtures into the throwaway project: rejected, since an absolute `Compile Include` reads them in place.
- Extending `scankit.Walk` to enter `testdata/`: rejected, since one caller does not justify a new option on the shared kit.

## Review rounds

_No rounds yet._

## Question ledger

- Which tier? Self-pick: `integration`, see the decision record.
- Which target framework? Self-pick: derived from `dotnet --version`.
- Do warnings fail the build? Self-pick: no.
