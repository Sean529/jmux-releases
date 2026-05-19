# jmux

A native macOS terminal built on [Ghostty](https://ghostty.org), with vertical tabs and a notification panel.

<!-- TODO: drop a hero screenshot or short GIF here -->
<p align="center">
  <img src="docs/screenshot.png" alt="jmux screenshot" width="800">
</p>

## Features

- **Vertical tabs** — fits long tab titles and dozens of sessions without horizontal scrolling.
- **Notification panel** — per-pane bottom status bar surfaces command-finished, OSC 9/4 progress, and renderer health.
- **Ghostty under the hood** — same GPU-accelerated renderer, same `~/.config/ghostty/config`.
- **Multi-workspace + split panes** — Bonsplit-powered drag/drop splits, sidebar workspaces, and undoable tab close.
- **Lossless remote sessions** — `jmux ssh <host>` ships a remote daemon for clean port-forwarding and a remote workspace bootstrap.
- **Agent skills** — `jmux skills install` writes bundled skills into Claude Code (`~/.claude/skills/`) and Codex (`$CODEX_HOME/skills/`).
- **Customizable shortcuts + i18n** — every cmux-owned shortcut is editable in Settings; UI is localized (English + Japanese to date).
- **Sparkle-based auto-update** — see [Auto-update](#auto-update) below.

## Download

Latest release: <https://github.com/Sean529/jmux-releases/releases/latest>

Grab `jmux-v<version>.zip` from the assets, unzip, and drag `jmux.app` to `/Applications`.

### First launch (Gatekeeper)

Builds are ad-hoc signed for local distribution, so macOS Gatekeeper blocks the first launch. Either:

- Right-click `jmux.app` → **Open** → confirm in the dialog, **or**
- Strip the quarantine flag once:
  ```bash
  xattr -dr com.apple.quarantine /Applications/jmux.app
  ```

Intel Macs are not covered by the published builds (arm64 only). On an Intel Mac you'd need to build from source with `ARCHS='arm64 x86_64' ONLY_ACTIVE_ARCH=NO`.

## Auto-update

jmux ships with [Sparkle](https://sparkle-project.org) and polls this repo for new versions:

- Appcast URL: `https://github.com/Sean529/jmux-releases/releases/latest/download/appcast.xml`
- The shipped app verifies the `ed25519` signature on every update before applying it, so a tampered appcast won't be accepted.

Once installed, you don't need to revisit this page — jmux checks on launch and prompts you when an update is available.

## License

See [LICENSE](./LICENSE).

## Source

The source repository is private. This repo only hosts the release artifacts (`.zip` + `appcast.xml`) consumed by Sparkle.
