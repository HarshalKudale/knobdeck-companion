# KnobDeck companions

Small add-ons that run *inside* other apps so [KnobDeck](https://knobdeck.harshalkudale.com) can
control them through a real interface instead of keyboard shortcuts. A KnobDeck plugin from
[knobdeck-plugins](https://github.com/Knobdeck/knobdeck-plugins) talks to each one: the editor
companions link to KnobDeck (or, for older KnobDeck versions, expose a local action server), and the
browser extension connects out to the browser plugin's local server.

This repo is the index. Every companion lives in its own repo with its own CI, versions and
one-click store releases; they're linked here as git submodules so they're all in one place.
Clone with `git clone --recurse-submodules`, or run `git submodule update --init` in an existing
clone.

| Companion | Repo | Runs in | KnobDeck plugin | Stores |
|---|---|---|---|---|
| KnobDeck VS Code Companion | [knobdeck-vscode-companion](https://github.com/Knobdeck/knobdeck-vscode-companion) ([`vscode/`](vscode)) | VS Code, VSCodium, Cursor, Windsurf | `vscode` | Visual Studio Marketplace, Open VSX |
| KnobDeck Companion for JetBrains IDEs | [knobdeck-intellij-companion](https://github.com/Knobdeck/knobdeck-intellij-companion) ([`intellij/`](intellij)) | IntelliJ IDEA, Android Studio, PyCharm, WebStorm and other JetBrains IDEs (2024.3+) | `intellij` | JetBrains Marketplace |
| KnobDeck Browser Companion | [knobdeck-browser-companion](https://github.com/Knobdeck/knobdeck-browser-companion) ([`browser/`](browser)) | Chrome, Edge, Brave and other Chromium browsers; Firefox, Zen | `browser-plugin` | Chrome Web Store (also for Edge, Brave), addons.mozilla.org |

- [PROTOCOL.md](PROTOCOL.md): how the plugins talk to the companions. Ids are a contract: add,
  never rename.
- [PRIVACY.md](PRIVACY.md): what the companions do with data (nothing leaves your computer).
- [`compat/`](compat): drives the real Rust plugins against a companion, the way the KnobDeck host
  does. Each companion's CI runs a copy of it (`test/compat/` or `test-support/compat/`); change it
  here first, then copy it over.

## Building, testing and releasing

See each repo's `CONTRIBUTING.md`. They all release the same way: bump the version, merge to
`master`, then **Actions → Release → Run workflow**. That runs CI, creates the tag `v<version>` and a
GitHub release with the packages, and publishes to the stores.

After a companion releases, point its submodule here at the new version:

```sh
git submodule update --remote <vscode|intellij|browser>
git commit -am "<companion>: <version>"
```
