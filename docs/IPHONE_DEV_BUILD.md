# Running Interview Lens on a physical iPhone

How the development build got onto a real iPhone on 2026-11-15, including each problem hit along the way and its fix. For the short in-class checklist, see [CLASS_QUICKSTART.md](CLASS_QUICKSTART.md).

**Result:** dev build installed on an iPhone 13 Pro. **On-device transcription works.**

---

## Why Expo Go isn't enough

### Two layers of a React Native app

```
┌─────────────────────────────────────────┐
│  JavaScript layer  (your src/ code)     │  ← Metro sends this over Wi-Fi/tunnel
│  screens, Redux, masking rules, fetch   │     and can change any time
├─────────────────────────────────────────┤
│  Native layer  (compiled app binary)    │  ← fixed when the app is BUILT
│  C++ / Swift / Obj-C, Apple frameworks, │     can't be changed over the network
│  permissions in Info.plist              │
└─────────────────────────────────────────┘
```

- **The JavaScript layer decides *what* to do:** screens, app logic, pattern (regex) masking, cloud requests.
- **The native layer *does* it on the phone:** microphone, files, Keychain, drawing the UI, running the JavaScript engine (Hermes), and fast compiled engines.
- **Where the AI lives:**
  - The *engines* are in the native layer: whisper.cpp (transcription), llama.cpp (masking), and Apple Speech (part of iOS).
  - The *model files* (Whisper base.en about 60 MB, Qwen about 1.1 GB) are downloaded data in the app's Caches folder. The engines load and run them.

**Expo Go is a native layer that someone else already built.** It can run our JavaScript, but it doesn't contain whisper.cpp, llama.cpp or expo-speech-recognition, and iOS doesn't let apps add native code later. So we build our own native layer, the **development build**.

Expo Go only contains the native modules Expo bundles into it. Interview Lens transcribes with **whisper.rn** and **expo-speech-recognition**, and masks with **llama.rn**. All three are native code, so in Expo Go the app runs in **demo mode** (sample transcripts, no real audio → transcript).

Real transcription needs the **development build**, our own app binary with those native modules compiled in.

| | Expo Go (`npx expo start --go`) | Dev build (`npx expo start`) |
| --- | --- | --- |
| Install | Expo Go from the App Store | `npx expo run:ios --device` (Xcode + signing) |
| QR code type | `exp://…` | `exp+interview-lens://…` |
| On-device transcription | ❌ demo only | ✅ |
| Opens in | Expo Go | the **Interview Lens** app |

---

## One-time setup (free Apple ID)

1. **Apple ID in Xcode:** Xcode app → Settings → Accounts → **+** → Apple ID. This creates a free *Personal Team*.
2. **Plug in the iPhone** with USB, unlock it, and tap **Trust This Computer**.
3. **Developer Mode on the iPhone:** Settings → Privacy & Security → **Developer Mode** → on → restart → **Turn On**.
4. **Signing team.** This is set in the **Xcode app**, not VS Code:
   ```bash
   open ios/InterviewLens.xcworkspace
   ```
   Blue **InterviewLens** project icon → under **TARGETS** pick **InterviewLens** → **Signing & Capabilities** → ☑ **Automatically manage signing** → **Team:** *(Personal Team)*. Xcode then creates the "Apple Development" certificate.
   - `npx expo prebuild --clean` regenerates `ios/` and **resets the team**. Set it again if that happens.
   - If the bundle ID is ever "not available", change `ios.bundleIdentifier` in `app.json` (not in Xcode), then run `npx expo prebuild --clean -p ios`.
5. **Stop repeated keychain prompts.** Run this once in a terminal and enter your **Mac login password** there:
   ```bash
   security set-key-partition-list -S apple-tool:,apple:,codesign: -s ~/Library/Keychains/login.keychain-db
   ```
6. **Build and install:**
   ```bash
   npx expo run:ios --device
   ```
   The first build takes about 10 minutes. Rebuilds take seconds to a few minutes. Keep the phone **unlocked**.
7. **Trust the developer on the iPhone:** Settings → General → **VPN & Device Management** → *Apple Development: …* → **Trust**.

## Every time after that

```bash
npx expo start --tunnel      # campus Wi-Fi
# or connect the Mac to the iPhone's Personal Hotspot and run: npx expo start
```

Open the **Interview Lens** app → tap the server (or scan the QR code with the Camera app). The first load over a tunnel may time out. Reload once.

## Free Apple ID limits

- The app stops opening **7 days** after install. Rerun `npx expo run:ios --device` with the phone plugged in to refresh it.
- **EAS cloud builds can't install on a device without the paid Apple Developer Program** ($99/yr). The free Xcode route above is the only free option.

---

## Problems hit and how they were fixed

| # | Symptom | Cause | Fix |
| --- | --- | --- | --- |
| 1 | QR code "not scannable" | (a) With `expo-dev-client` installed, the default QR code is an `exp+interview-lens://` link for the dev build, which wasn't on the phone yet. (b) VS Code's terminal can draw QR blocks badly. | Use `npx expo start --go --tunnel` for Expo Go. For rendering, make the terminal bigger, use Terminal.app, or type the `exp://` URL into Expo Go. |
| 2 | Expo Go: "network connection established… **Gateway timeout**" | The first Metro bundle took longer than the tunnel's timeout. | Wait for `iOS Bundled` in the terminal, then **Reload**. Fixed it. |
| 3 | App stuck in **demo mode** | Expo Go can't load whisper.rn or expo-speech-recognition. | Build and install the dev build (setup above). |
| 4 | Not sure where "signing team" is | It's in the **Xcode app**, not VS Code. | `open ios/InterviewLens.xcworkspace` → target → Signing & Capabilities. |
| 5 | iPhone listed as "unavailable" | **Developer Mode was disabled**, and there was no signing certificate yet. | Turned on Developer Mode and set the team in Xcode. |
| 6 | macOS keeps asking for the **"login" keychain password**, even after *Always Allow* | That's the **Mac login password**. `codesign` runs once per embedded framework (10 + the app), and the key's partition list didn't include Apple's tools. | Enter the password each time for that build. Run `security set-key-partition-list …` (step 5) to stop it for good. |
| 7 | Build succeeded, install failed: **`ApplicationVerificationFailed`** | Two frameworks (`ExpoModulesWorklets`, `ReactNativeDependencies`) were **left unsigned** because their keychain prompts were never approved. Found with `codesign -dv` on each framework. | Ran the partition-list fix, then rebuilt. Every framework was signed with no prompts, and the rebuild took about 20 s. |
| 8 | Installed, but launch denied: *"invalid code signature… profile has not been explicitly trusted by the user"* | iOS requires the user to trust a free developer certificate. | Settings → General → VPN & Device Management → **Trust**. |

### Handy diagnostic commands

```bash
xcrun devicectl list devices                                   # is the phone connected / available?
xcrun devicectl device info details --device <UDID> | grep -i "developer mode"
security find-identity -v -p codesigning                       # any signing certificate?
grep DEVELOPMENT_TEAM ios/InterviewLens.xcodeproj/project.pbxproj  # team set on the project?
# check every embedded framework is signed:
for f in ~/Library/Developer/Xcode/DerivedData/InterviewLens-*/Build/Products/Debug-iphoneos/InterviewLens.app/Frameworks/*.framework; do
  echo "$(basename $f): $(codesign -dv "$f" 2>&1 | grep -E 'TeamIdentifier|not signed')"; done
xcrun devicectl device info apps --device <UDID> | grep -i interview   # is the app installed?
```
