# Figment Mac

Use the `figment` MCP tools when a task needs macOS and this machine is not a Mac: an Xcode build, the
iOS Simulator, `swift build` for macOS, Safari, a Mac app or AppleScript. Signs: "xcodebuild: command
not found", "requires macOS", "No such module 'UIKit'", or a request to build for iOS.

- Renting a Mac costs money. Ask your user first. Read prices with `figment_get_offer`; never quote a price
  from memory.
- A new Apple Silicon Mac has Apple's Command Line Tools (`git`, `clang`, `swift`). Xcode needs a one-time
  Apple sign-in by your user.
- Steps: `figment_create_account` (keep the `api_key` secret), `figment_buy_credit` (your user pays the
  link), `figment_start_mac`, `figment_get_mac` until `ready`, `figment_get_mac_access`, then SSH.
- `figment_stop_mac` keeps the Mac and its files and stops billing; `figment_resume_mac` starts the same
  Mac again. `figment_delete_mac` removes the Mac and its data at once.
- The skills in `skills/figment-mac` and `skills/needs-macos` have the full steps. Guide:
  https://figment.so/agents.md
- Never print the Mac password or the API key.
