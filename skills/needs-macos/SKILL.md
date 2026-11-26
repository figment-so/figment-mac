---
name: needs-macos
description: Use when a task needs macOS and this machine runs Linux or Windows. Signs - a command fails with "xcodebuild: command not found", "xcrun: command not found", "No such module 'UIKit'", "requires macOS", "iOS builds are supported only on macOS hosts" or "CocoaPods is not supported on this platform"; or the user asks to build, test or archive an iOS or macOS app, run the iOS Simulator, run react-native run-ios, flutter build ios, pod install, or swift build for macOS. Says what a rented Figment Mac can run at once and what needs Xcode, then how to rent one, copy the project over SSH, build, and bring the results back.
license: MIT
metadata:
  author: Figment
  homepage: https://figment.so/agents.md
---

# The task needs macOS, and this machine is not a Mac

Xcode, the iOS and macOS SDKs, the iOS Simulator, `codesign` and `notarytool` run only on macOS.
There is no supported way to run them on Linux or Windows. Do not try to install Xcode here, and do
not download an SDK from a source that is not Apple.

## 1. Check the machine

Run `uname -s`.

- `Darwin`: this is a Mac, and this skill does not apply. If Xcode is missing, your user installs it
  from the App Store; `xcode-select --install` installs only the Command Line Tools.
- `Linux`, `MINGW*`, `MSYS*` or `CYGWIN*`, or Windows (`$env:OS` is `Windows_NT`): continue.

## 2. Find what the task needs on the Mac

| The task | What the Mac needs |
| --- | --- |
| `swift build` and `swift test` for macOS, `clang`, `git`, `python3`, scripts that need macOS | Apple's Command Line Tools. A new Apple Silicon Mac from Figment has them. |
| An iOS or Mac app with `xcodebuild`, the iOS Simulator, SwiftUI previews, CocoaPods, `flutter build ios`, `react-native run-ios`, Expo local iOS builds | Xcode. Your user installs it one time, because Apple asks a person to sign in. |
| Sign with a team, install on a device, upload to TestFlight or the App Store | Xcode and your user's Apple Developer account. Your user does these steps. |

## 3. Ask your user first

Renting a Mac costs money. Tell your user:

- why the task needs macOS (quote the error),
- the plan and its hourly price, read now from `https://figment.so/api/agent/v1/offer` (never quote a
  price from memory),
- that setup is not instant,
- that a task with Xcode needs them to sign in to Apple one time (the guide section "Install Xcode").

Continue only when they agree. If they have a Mac, they can run the step there instead.

Plan: `apple-silicon-vm-8gb` for a task with Xcode (more memory and storage), `apple-silicon-vm` for
Command Line Tools work. `intel-vm` only after your user reads its compatibility notice in the
offer. Check `available_now` in the offer.

## 4. Rent the Mac and connect

Use the `figment-mac` skill of this plugin, or the MCP tools `figment_*` if the server
`https://figment.so/api/mcp` is connected. They cover the account, the payment link for your user,
the order, the wait for `ready`, the access call and the SSH key. SSH steps that keep the password out
of logs: "Use SSH from an agent" in `https://figment.so/agents.md`.

## 5. Copy the project, build, bring the results back

After the key is installed (`HOST`, `PORT`, `MAC_USER`, `key` and `known_hosts` as in the guide):

```bash
mac() { ssh -i key -o IdentitiesOnly=yes -o BatchMode=yes -o UserKnownHostsFile="$PWD/known_hosts" -p "$PORT" "$MAC_USER@$HOST" "$@"; }
tar -czf - --exclude=.git --exclude=node_modules -C /path/to/project . | mac 'mkdir -p ~/work && tar -xzf - -C ~/work'
mac 'cd ~/work && swift build -c release'
mkdir -p mac-output && mac 'cd ~/work && tar -czf - .build/release' | tar -xzf - -C mac-output
```

With Xcode installed:

```bash
mac 'xcrun simctl list devices available'
mac 'cd ~/work && xcodebuild -scheme App -destination "generic/platform=iOS Simulator" CODE_SIGNING_ALLOWED=NO build'
mac 'cd ~/work && xcodebuild test -scheme App -destination "platform=iOS Simulator,name=<a device from the list>"'
```

- `xcodebuild -downloadPlatform iOS` gets the iOS platform when the Simulator is missing.
- Look at the free disk space before a large install: `mac 'df -h /'`.
- Send the build log back with the results when a build fails.

## 6. In CI

In GitHub Actions, the action `figment-so/run-on-mac` runs one job step on a rented Mac and ends the
Mac after the step, also when the job fails or is cancelled. The Mac has the Command Line Tools and
no Xcode.

## 7. Finish

- Copy back the files that you need first.
- Then end or stop the Mac as the `figment-mac` skill says, and tell your user what it cost
  (`spent_cents`). A Mac that runs is billed, also when it is idle.
- The Mac password and the API key are secrets. Do not print them in chat, logs or commits.
- Tell Figment how it went, or what was missing: `POST https://figment.so/api/agent/v1/feedback` with
  `{"message":"..."}`. No key is needed.
