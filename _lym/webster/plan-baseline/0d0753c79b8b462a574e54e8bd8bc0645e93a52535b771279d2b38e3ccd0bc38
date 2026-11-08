# Card 2 — socket-mode-client

**Create:**
- `plan:internal/slackengine#Socket` -> `type Socket interface`
- `plan:internal/slackengine#Envelope` -> `type Envelope struct`
- `plan:internal/slackengine#Dialer` -> `type Dialer func(ctx context.Context) (Socket, string, error)`
- `plan:internal/slackengine#NewDialer` -> `func NewDialer(baseURL, appToken string, client *http.Client, helloTimeout time.Duration) Dialer`
- `plan:internal/slackengine#HelloTimeout` -> `const HelloTimeout = 10 * time.Second`
- `plan:internal/slackengine#wsSocket` -> `type wsSocket struct`
- `plan:internal/slackengine#wsSocket.Read` -> `func (s *wsSocket) Read(ctx context.Context) (Envelope, error)`
- `plan:internal/slackengine#wsSocket.Ack` -> `func (s *wsSocket) Ack(ctx context.Context, envelopeID string) error`
- `plan:internal/slackengine#wsSocket.Ping` -> `func (s *wsSocket) Ping(ctx context.Context) error`
- `plan:internal/slackengine#wsSocket.Close` -> `func (s *wsSocket) Close() error`
- `plan:internal/slackengine_test#TestSocketClient` -> `func TestSocketClient(t *testing.T)`

**Edit:**
- `go.mod`
- `go.sum`

**Uses:**
- `plan:internal/slackengine#API.OpenConnection`
- `internal/slackengine#NewAPI`
- `internal/slackengine_test#answerJSON`

**Intent:**
Add the Socket Mode client over `github.com/coder/websocket` (D11), required at `v1.8.15`, the newest version the Go proxy lists on 2026-11-08; it pulls in no further module.
The implementer runs `go get github.com/coder/websocket@v1.8.15`, which writes both `go.mod` and `go.sum`.

`Socket` is the seam the relay is told, as it is told its `SlackClient`:
`Read(ctx) (Envelope, error)` returns the next envelope, `Ack(ctx, envelopeID) error` writes `{"envelope_id": …}`,
`Ping(ctx) error` sends a WebSocket ping and returns once the pong arrives or ctx ends, and `Close() error` closes the connection.
`Envelope` holds `Type` (`hello`, `events_api`, `disconnect`), `EnvelopeID`, `Reason` (a `disconnect`'s reason),
and for an `events_api` envelope the event's `Channel`, `TS`, `ThreadTS` and `Subtype`, read from `payload.event`.

`NewDialer` returns a `Dialer` that builds an `API` with the app-level token, calls `OpenConnection`, dials the answered URL with `websocket.Dial` over client,
and reads the first message, which must be a `hello`, within `helloTimeout`; it returns the socket and the hello's `connection_info.app_id`.
A connection that sends no `hello` within that bound is closed and the dial is a plain error, so a connection gone half-open before the `hello` never hangs the holder, whose ping runs only while it serves.
Production passes `HelloTimeout`, ten seconds, and a test passes a short bound.
A refused `apps.connections.open` comes back as its `*APIError`, so a caller tells a refused or disabled app from a transport failure;
a first message that is no `hello` closes the connection and is a plain error.
`wsSocket` runs one reader goroutine that decodes each text message into the channel `Read` takes from,
because coder/websocket answers a ping's pong only while a read is in progress; the goroutine ends, and `Read` returns its error, once the connection fails or closes.

Tests, tier0: `TestSocketClient` runs the real client against an `httptest` server that answers `apps.connections.open` with its own `ws://` URL and accepts the WebSocket with `websocket.Accept`.
One row has the server send a `hello` with an app id, an `events_api` envelope for channel `C1` and a `disconnect`;
it asserts the dial's app id, the envelope's decoded fields, that the server received `{"envelope_id": …}` for the events envelope's id after `Ack`, that `Ping` returns nil, and that the `disconnect` reads with its reason.
A second row has `apps.connections.open` answer `{"ok":false,"error":"invalid_auth"}` and asserts the dial returns an `*APIError` with that code.
A third has the first message be an `events_api` envelope and asserts the dial fails.
A fourth has the server accept the WebSocket and send nothing, dialed with a 50-millisecond bound, and asserts the dial fails once the bound passes.

**ImpactSummary:** adds one direct module to `go.mod` and its sums to `go.sum`; nothing else in the tree imports the new client yet.
