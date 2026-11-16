# Class quick-start — run Interview Lens on my iPhone

A short checklist for running the app in class. The full setup and troubleshooting history is in [IPHONE_DEV_BUILD.md](IPHONE_DEV_BUILD.md).

> **The app is already installed on my iPhone (dev build, installed 2026-11-15).** You don't need to rebuild it. Just start Metro and open the app.
> The free Apple ID signature **expires around 2026-11-22**. After that, rebuild (see "If the app won't open" below).

---

## Before class (2 min)

- [ ] iPhone charged, **Interview Lens** icon still on the home screen
- [ ] Mac charged, project open in VS Code
- [ ] Bring the **USB cable** (only needed if a rebuild is required)
- [ ] Optional: phone on Wi-Fi so it can reach the tunnel quickly

## In class (3 steps)

**1. Start Metro in the VS Code terminal** (project folder):

```bash
npx expo start --tunnel
```

- Use **`npx`**, not `npm`.
- **No `--go`**. `--go` is only for the Expo Go demo mode.
- `--tunnel` is needed on campus Wi-Fi. MWireless blocks phone ↔ Mac traffic, and iOS blocks loading the app from a public IP.
- Wait for the QR code and the line `› Metro waiting on …`.

**2. Optional: compile the bundle first** so the phone doesn't time out. Run this in a **second terminal**:

```bash
curl -s -o /dev/null "http://localhost:8081/node_modules/expo-router/entry.bundle?platform=ios&dev=true&minify=false" && echo warmed
```

**3. Open the app on the iPhone.** Tap the **Interview Lens** icon (not Expo Go), then tap the listed server, or scan the QR code with the **Camera** app.

✅ It's working if there's no demo banner and **Settings** shows **"Development build"**.

---

## If something goes wrong

| What you see | Fix |
| --- | --- |
| **Gateway timeout** on first open | Normal on a tunnel's first load. Wait for `iOS Bundled …` in the terminal, then shake the phone → **Reload** (or reopen the app). |
| `Port 8081 is being used` | Another Metro is running. Close the other terminal, or run `lsof -ti :8081 \| xargs kill`, then start again. |
| QR code won't scan | Make the VS Code terminal bigger, or run the command in macOS **Terminal.app**. Or just open the app and tap the server, so you don't need the QR. |
| Tunnel keeps failing | Turn on iPhone **Personal Hotspot**, connect the Mac to it, and run `npx expo start` (no `--tunnel`). |
| Red error screen in the app | Read the terminal. Then try `npx expo start --tunnel --clear`. |
| App shows **demo mode** | You opened it in **Expo Go**. Open the **Interview Lens** app icon instead. |
| Whisper model missing | iOS may clear cached models. Settings → On-device models → download Whisper base.en again (needs Wi-Fi), or use Apple on-device speech. |
| **App won't open / "Untrusted Developer"** | The 7-day signature expired. Plug in the iPhone (unlocked) and run `npx expo run:ios --device`. Then Settings → General → VPN & Device Management → **Trust**. |

## Last resort: Expo Go demo

If the dev build can't connect at all, show the demo mode in Expo Go (sample audio and transcripts, **no real on-device transcription**):

```bash
npx expo start --go --tunnel
```

Scan with the Camera app → **Open in Expo Go**.
