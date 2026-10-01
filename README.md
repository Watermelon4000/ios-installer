# iOS Installer

Build an Xcode project and put it on your own iPhone or iPad **over Wi-Fi**, then open it — one command, no
cable, no tunnel, no QR code, no TestFlight.

```bash
cd MyApp
ios-installer
```

```
==> Generating the Xcode project (xcodegen)
==> Building MyApp (Debug) for devices
    signing through App Store Connect API key ABC123DEFG
==> Finding the device
    My iPhone (1A2B3C4D-0000-0000-0000-000000000000)
==> Installing com.example.myapp
==> Opening
==> Done
```

## How it works

Xcode 15 and later ship `devicectl`, which talks to any device that has been **paired with this Mac once**
over the local network. This script wraps the whole loop around it:

1. finds the project (runs `xcodegen` if there is a `project.yml`; prefers a `.xcworkspace`), the scheme and
   builds a signed device build with `xcodebuild`;
2. picks the first paired iPhone / iPad that is reachable (or the one you name);
3. installs it, waiting while the device is locked; then opens the app.

Over-the-air tools that serve an `itms-services://` link need a public HTTPS tunnel, which proxies and
firewalls often break. Here nothing leaves the local network.

## Requirements

- macOS with Xcode 15 or later (the command-line tools are enough — the Xcode app itself never has to open).
- The device: paired with this Mac once (connect by cable, tap **Trust**), **Developer Mode** on, on the same
  Wi-Fi as the Mac, unlocked when installing.
- The device registered on your Apple developer team (true for any device Xcode has run an app on).
- `python3` (ships with the command-line tools); `xcodegen` only if your project uses `project.yml`.

## Install

```bash
git clone https://github.com/Watermelon4000/ios-installer.git
ln -s "$PWD/ios-installer/ios-installer" /usr/local/bin/ios-installer   # or anywhere on PATH
```

## Signing without opening Xcode

If Xcode has your Apple account signed in, nothing to set up. Otherwise (or if the Xcode app will not launch,
say on a new macOS), use an **App Store Connect API key with the Admin role**: `xcodebuild` then registers the
App ID and creates the development profile itself.

1. App Store Connect → Users and Access → Integrations → App Store Connect API → generate a key (Admin).
2. Put `AuthKey_<KEYID>.p8` in `~/.appstoreconnect/private_keys/` (`chmod 600` it).
3. Write `~/.config/ios-installer/config`:

```bash
ASC_KEY_ID=ABC123DEFG
ASC_ISSUER_ID=00000000-0000-0000-0000-000000000000
# ASC_KEY_PATH=~/.appstoreconnect/private_keys/AuthKey_ABC123DEFG.p8   # the default
```

An Admin key can do a lot on your account: keep it out of repositories, revoke it if it leaks.

## Options

```
ios-installer --device "iPad"        a device by (part of) its name, or its identifier
ios-installer --app build/X.app      install an existing device build, skip building
ios-installer --scheme MyApp --configuration Release
ios-installer --workspace MyApp.xcworkspace
ios-installer --no-launch            install only
ios-installer --list                 paired devices and whether they can be reached
```

## When it fails

| Message | Fix |
|---|---|
| `No reachable paired device` | Same Wi-Fi? Unlocked? Paired once by cable? Developer Mode on? `--list` shows what the Mac sees. |
| `Signing failed` / `No profiles for …` | Sign in to Xcode, or set the API key above (it needs the **Admin** role; a lower role gives `401`). |
| `the device refused the app's signature` | The device is not on the team's profile — run once from Xcode with the device attached, or register its UDID on developer.apple.com. |
| `is a Simulator build` | `--app` must point at an `…-iphoneos/…app`, not `…-iphonesimulator/…`. |

## For AI coding agents

[`SKILL.md`](SKILL.md) tells Claude Code, Antigravity and similar agents when and how to use this.

## License

MIT
