# Companion protocol

How KnobDeck's plugins talk to the companions. Every companion speaks the same small HTTP dialect,
so a new one (for another editor or app) can reuse the Rust side unchanged.

| Companion | KnobDeck plugin | Default port | Settings in the plugin's `.env` |
|---|---|---|---|
| VS Code ([knobdeck-vscode-companion](https://github.com/Knobdeck/knobdeck-vscode-companion)) | `vscode` | `32145` | `VSCODE_HOST`, `VSCODE_PORT`, `VSCODE_PASSWORD` |
| IntelliJ ([knobdeck-intellij-companion](https://github.com/Knobdeck/knobdeck-intellij-companion)) | `intellij` | `21420` | `IDEA_HOST`, `IDEA_PORT`, `IDEA_PASSWORD` |
| Browser ([knobdeck-browser-companion](https://github.com/Knobdeck/knobdeck-browser-companion)) | `browser-plugin` | `32147` | `BROWSER_PORT` (see [Browser extension](#browser-extension): the direction is reversed) |

## Transport

- HTTP/1.1 on `127.0.0.1` only. Every request is a `GET` (except the VS Code window handoff).
- **Password**: when one is set, the request carries it verbatim in `Authorization` (no scheme).
  A wrong or missing password is `401` (VS Code) or `403` (IntelliJ); KnobDeck treats both as
  "wrong password" and asks the user to run setup again.
- **Web pages are refused** with `403`: any request with an `Origin` or `Sec-Fetch-*` header, or a
  `Host` other than `127.0.0.1` / `localhost` / `[::1]` on the server's own port. KnobDeck sends
  none of these. This keeps a website from driving the editor through the user's browser, even
  with no password set.
- Responses are JSON. Errors are `{"ok": false, "error": "<message for the user>"}`; KnobDeck shows
  the message on the button.

## Endpoints

| Request | Answer |
|---|---|
| `GET /health` | `200 {"ok": true, "app": "...", "version": "...", ...}`. Setup calls this to check the connection. |
| `GET /api/action/<id>[?name=…\|?uri=…]` | `200 {"ok": true}` once the action ran; `200 {"ok": true, "pending": true}` (VS Code) or `202` (IntelliJ) if it is still running or queued after ~5 s; `400` missing parameter; `404` unknown action / task / configuration / terminal; `409` not possible right now (e.g. Step Over with no debug session); `500` anything else. `<id>` is percent-encoded. |
| `GET /api/query/<name>` | `200 {"items": [{"value": "...", "label": "..."}]}` |

### VS Code action ids

`vscode.openCommandPalette`, `vscode.quickOpen`, `vscode.explorer`, `vscode.search`,
`vscode.sourceControl`, `vscode.extensions`, `vscode.toggleTerminal`, `vscode.problems`,
`vscode.saveAll`, `vscode.formatDocument`, `vscode.toggleSidebar`, `vscode.togglePanel`,
`vscode.zenMode`, `vscode.splitEditor`, `vscode.newTerminal`, `vscode.stopDebugging`;
`vscode.runTask?name=`, `vscode.startDebugging?name=` (launch configuration or compound),
`vscode.focusTerminal?name=`, `vscode.openEditor?uri=`. Any other id runs that VS Code command
with no arguments (KnobDeck's Custom Command).

Queries: `tasks`, `launch-configs`, `terminals`, `open-editors`.

**Several windows.** Each VS Code window runs the companion, and one holds the port. A window that
gains focus sends `POST /api/handoff` with `X-KnobDeck-Companion: handoff` (and the password); the
holder answers `200` and closes its server, and the focused window binds the port.

### IntelliJ action ids

Any IDE action id (`SearchEverywhere`, `StepOver`, …; **Tools → KnobDeck: Copy an Action ID…** lists
them). `Run?name=<configuration>` / `Debug?name=<configuration>` start that run configuration in the
last focused project; without `name` they act like the toolbar buttons. `KnobDeck.ShowActionBrowser`
opens the action ID browser (`streamdeck.show.action.browser`, sent by early KnobDeck builds, is an
alias).

Queries: `run-configs`.

## Companion hub (current KnobDeck)

Current KnobDeck runs one **companion hub** that companions dial instead of opening a port:
KnobDeck's own contract, specified in
[knobdeck-sdk/knobdeck-protocol/REFERENCE.md](https://github.com/Knobdeck/knobdeck-sdk/blob/master/knobdeck-protocol/REFERENCE.md)
and implemented by the [`@knobdeck/companion`](https://www.npmjs.com/package/@knobdeck/companion)
client. The VS Code companion (1.1+) links there first; the HTTP dialect above stays for older
KnobDeck versions. On the hub, the VS Code companion takes the same action ids as
`/api/action/<id>` (parameters as an object, e.g. `{"name": "build"}`), `query` with `{"name": …}`
for the lists, and sends a `window_focused` event when its window gains focus.

## Browser extension

The one companion that dials the plugin instead of listening, because a browser extension can't open
a port. `browser-plugin` listens on `127.0.0.1:32147` (`BROWSER_PORT`); each browser's extension
connects with a WebSocket and the plugin lists the connected ones as the Browser choices.

- **Web pages are refused**: the handshake needs an `Origin` of `chrome-extension://…` or
  `moz-extension://…` and a `Host` of `127.0.0.1` / `localhost` / `[::1]` on that port (`403`
  otherwise). No password: nothing but a local program that fakes an extension origin can connect.
- Messages are JSON text frames.
  - Extension → plugin, first: `{"type":"hello","protocol":1,"id":"<per-profile uuid>","browser":"chrome|msedge|brave|firefox|zen|opera","name":"Chrome","family":"chromium|firefox","version":"1.0.0"}`.
    The plugin answers `{"type":"ready","protocol":1}`. A reconnect with the same `id` replaces the old link.
  - Extension → plugin, every 20 s: `{"type":"ping"}`, answered `{"type":"pong"}`. The plugin drops a link silent for 75 s.
  - Plugin → extension: `{"id":<n>,"action":"<id>","params":{…}}`. Answer: `{"id":<n>,"ok":true,"result":{…}}`,
    or `{"id":<n>,"ok":false,"error":"<message for the user>"}`, or `{"id":<n>,"ok":false,"unsupported":true}`
    when the browser has no API for it (the plugin may then send a shortcut instead).
- Actions: the plugin's action ids `new-tab`, `new-window`, `new-private-window`, `close-tab`,
  `reopen-closed-tab`, `next-tab`, `previous-tab`, `reload-tab`, `hard-reload-tab`, `go-back`,
  `go-forward`, `stop-loading`, `print-page`, `toggle-fullscreen`, `bookmark-page`, `open-downloads`,
  `open-history`, `open-bookmarks-manager`; `open-url` (`url`), `custom-keybind` (`keys`, KnobDeck key
  names), `zoom-set` / `zoom-by` (`percent`; answer `{"percent": n}`), `bookmarks` / `history`
  (answer `{"items":[{"title","url"}]}`, 16 newest). Always on the active tab of the last used window.
- `focus-address-bar`, `find-in-page` and `save-page` are answered `unsupported` by every browser.

## Compatibility

Ids and query names are a contract with released KnobDeck plugins: add, never rename or remove.
Each companion's CI drives the real Rust plugin against it with a copy of [`compat/`](compat).
