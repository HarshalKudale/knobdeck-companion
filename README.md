# KnobDeck companions

Small add-ons that run *inside* other apps so [KnobDeck](https://knobdeck.harshalkudale.com) can
control them through a real interface instead of keyboard shortcuts. Each one exposes a local action
server that a KnobDeck plugin from [knobdeck-plugins](https://github.com/Knobdeck/knobdeck-plugins)
talks to.

Companions that have grown up live in their own repos (own CI, versions and store publishing) and
are linked here as git submodules, so this repo shows them all in one place. Clone with
`git clone --recurse-submodules`, or run `git submodule update --init` in an existing clone.

| Companion | Runs in | KnobDeck plugin | Store |
|---|---|---|---|
| [`vscode/`](https://github.com/Knobdeck/knobdeck-vscode-companion) — KnobDeck VS Code Companion (submodule) | VS Code, VSCodium, Cursor, Windsurf | `vscode` | Visual Studio Marketplace, Open VSX |
| [`intellij/`](intellij) — KnobDeck Companion | IntelliJ IDEA, Android Studio, PyCharm, WebStorm and other JetBrains IDEs (2024.3+) | `intellij` | JetBrains Marketplace |

- [PROTOCOL.md](PROTOCOL.md): the HTTP interface the plugins rely on. Ids are a contract: add,
  never rename.
- [PRIVACY.md](PRIVACY.md): what the companions do with data (nothing leaves your computer).
- [`compat/`](compat): drives the real Rust plugins against the companions, the way the KnobDeck
  host does. CI builds the plugins from `knobdeck-plugins` and runs it on every change.

## Build and test

**VS Code**: see [CONTRIBUTING.md](https://github.com/Knobdeck/knobdeck-vscode-companion/blob/master/CONTRIBUTING.md)
in its repo. It carries a copy of `compat/driver.mjs` in `test/compat/`; keep the two in step.

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

**VS Code** releases from its own repo (tag `v<version>`, Trusted Publishing to the Visual Studio
Marketplace and Open VSX); see its CONTRIBUTING.md. After a release, point the submodule at it:
`git -C vscode pull origin master && git commit -am "vscode: <version>"`.

**IntelliJ**: bump `pluginVersion` in `intellij/gradle.properties`, add a changelog entry, merge to
`master`, then push a tag `intellij-v<version>`. CI tests, builds, creates a GitHub release with the
package, and publishes to JetBrains Marketplace when its secrets are set:

| Secret | For |
|---|---|
| `JETBRAINS_MARKETPLACE_TOKEN` | JetBrains Marketplace |
| `JETBRAINS_CERTIFICATE_CHAIN`, `JETBRAINS_PRIVATE_KEY`, `JETBRAINS_PRIVATE_KEY_PASSWORD` | Plugin signing |
