# KnobDeck companions

Small add-ons that run *inside* other apps so [KnobDeck](https://knobdeck.harshalkudale.com) can
control them through a real interface instead of keyboard shortcuts. Each one exposes a local action
server that a KnobDeck plugin from [knobdeck-plugins](https://github.com/Knobdeck/knobdeck-plugins)
talks to.

| Companion | Runs in | KnobDeck plugin | Store |
|---|---|---|---|
| [`vscode/`](vscode) — KnobDeck VS Code Companion | VS Code, VSCodium, Cursor, Windsurf | `vscode` | Visual Studio Marketplace, Open VSX |
| [`intellij/`](intellij) — KnobDeck Companion | IntelliJ IDEA, Android Studio, PyCharm, WebStorm and other JetBrains IDEs (2024.3+) | `intellij` | JetBrains Marketplace |

- [PROTOCOL.md](PROTOCOL.md): the HTTP interface the plugins rely on. Ids are a contract: add,
  never rename.
- [PRIVACY.md](PRIVACY.md): what the companions do with data (nothing leaves your computer).
- [`compat/`](compat): drives the real Rust plugins against the companions, the way the KnobDeck
  host does. CI builds the plugins from `knobdeck-plugins` and runs it on every change.

## Build and test

**VS Code** (Node 20+):

```sh
cd vscode
npm ci
npm test                       # protocol tests with a stand-in vscode module
npm run test:e2e               # in a real VS Code (downloads it; needs a display)
npx vsce package --no-dependencies --skip-license
```

Point `KNOBDECK_VSCODE_PLUGIN` / `KNOBDECK_VSCODE_SETUP` at a built `vscode` / `vscode-setup` from
knobdeck-plugins (`cargo build -p vscode`) to drive them in both test runs.

**IntelliJ** (JDK 21):

```sh
cd intellij
./gradlew test                 # headless IDE tests
./gradlew buildPlugin          # build/distributions/*.zip
./gradlew verifyPlugin         # JetBrains Plugin Verifier against 2024.3, 2025.3, 2026.3
./gradlew runIde               # try it in a sandbox IDE
```

`KNOBDECK_INTELLIJ_PLUGIN` / `KNOBDECK_INTELLIJ_SETUP` drive the real `intellij` plugin in `test`
(needs Node).

## Releasing

Bump the version (`vscode/package.json`, or `pluginVersion` in `intellij/gradle.properties`), add a
changelog entry, merge to `master`, then push a tag `vscode-v<version>` or `intellij-v<version>`.
CI tests, builds, creates a GitHub release with the package, and publishes to the stores when their
secrets are set:

| Secret | For |
|---|---|
| `VSCE_PAT` | Visual Studio Marketplace (publisher `knobdeck`) |
| `OVSX_PAT` | Open VSX (namespace `knobdeck`) |
| `JETBRAINS_MARKETPLACE_TOKEN` | JetBrains Marketplace |
| `JETBRAINS_CERTIFICATE_CHAIN`, `JETBRAINS_PRIVATE_KEY`, `JETBRAINS_PRIVATE_KEY_PASSWORD` | Plugin signing |
