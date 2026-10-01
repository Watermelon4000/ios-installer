---
name: ios-installer
description: >-
  Build an Xcode project and install it on the user's own paired iPhone or iPad over Wi-Fi, then open it,
  with the `ios-installer` script — no cable, tunnel, QR code or TestFlight. Use when the user wants the
  current build "on my phone", "on my iPad", "装到手机上", "无线安装", or to try a change on a real device,
  and the device is theirs (paired with this Mac). Not for Simulator runs, not for other people's devices
  (use an over-the-air link or TestFlight), not for App Store submission.
---

# Installing on the user's own device over Wi-Fi

`ios-installer` builds the project in the current folder, finds a paired, reachable iPhone / iPad with
`devicectl`, installs the app and opens it. It never leaves the local network.

## Run it

From the project root (the folder with the `.xcodeproj`, `.xcworkspace` or `project.yml`):

```bash
ios-installer
```

- Many devices: `ios-installer --device "iPhone"` (part of the name) — `ios-installer --list` shows them.
- A build already made: `ios-installer --app path/to/Debug-iphoneos/App.app`.
- The scheme is the project's name unless `--scheme` says otherwise.

The build runs for a minute or more; that is normal. In sandboxed agent shells, `devicectl` and Apple's
signing services usually need the command to run outside the sandbox — request that permission for this
command rather than working around it.

## Before you blame the script

- **"No reachable paired device"**: ask the user to unlock the device and check it is on the same Wi-Fi as
  the Mac. A device never paired needs one cable connection and "Trust" — the user has to do that.
- **Locked device**: the script waits (120 s by default) and says so; tell the user to unlock it.
- **Signing errors ("No Accounts", "No profiles for …")**: Xcode has no account. The fix is an App Store
  Connect API key with the **Admin** role in `~/.config/ios-installer/config` (`ASC_KEY_ID`,
  `ASC_ISSUER_ID`; key file in `~/.appstoreconnect/private_keys/`). A `401` means the key's role is too low
  or it was revoked. Never print, copy or commit the `.p8` file's contents.
- **The device refuses the signature**: it is not on the team's provisioning profile. The user registers it
  (run once from Xcode with it attached, or add the UDID on developer.apple.com).

## Report back

Say which device it went to and that the app was opened (or why not). Don't claim success unless the script
printed `Done`.
