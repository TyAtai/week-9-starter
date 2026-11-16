# Build process log — Interview Lens

**Project:** UX research interview app (Expo / React Native, TypeScript) built from `UX_Research_App_TODO.md`, referencing the course demo [UMSI669/Voice-To-Text](https://github.com/UMSI669/Voice-To-Text).
**Date:** Monday, 2026-11-09; session 2 (physical iPhone) on Sunday, 2026-11-15 — see §4. All times local, America/Detroit.
**Repository:** https://github.com/TyAtai/week-9-starter

## How these times were measured

- **Exact** times come from git commits (`git log`) and build logs.
- **`~` times** come from file creation/modification times and screenshots, accurate to about a minute.
- Durations are differences between those times.

---

## 1. Detailed timeline

### Summary

| Phase | Start | End | Duration |
| --- | --- | --- | --- |
| Repo setup (manual) | Nov 8, 20:51 | Nov 9, 10:16 | — (outside the AI session) |
| Research: spec, reference repo, Expo/library docs, model selection | ~10:42 | ~10:49 | ~7 min |
| Synthetic demo audio | ~10:49 | ~10:50 | ~1 min |
| Core services (models, privacy, analysis, storage, transcription, store) | ~10:50 | ~11:00 | ~10 min |
| Unit tests + fixes | ~11:00 | ~11:03 | ~3 min |
| UI screens | ~11:03 | ~11:11 | ~8 min |
| Lint/type fixes, iOS bundle check | ~11:11 | ~11:12 | ~1 min |
| Expo Go simulator verification | ~11:12 | ~11:17 | ~5 min |
| README + first commit/push | ~11:17 | 11:22:14 | ~5 min |
| Run demo for user | ~11:19 | ~11:21 | ~2 min |
| Development build (fail → fix → success) | 11:32:22 | ~11:39 | ~7 min |
| Docs update + commit/push (with rebase) | ~11:40 | 11:40:54 (commit) | ~2 min |
| Terminal/QR troubleshooting, tunnel fix | ~11:45 | 11:49:58 | ~5 min |
| This process document | ~11:52 | — | — |

**AI session total:** about 1 hour 10 minutes, from ~10:42 to ~11:52.

### Timestamped log

| Time | Milestone / event | Notes |
| --- | --- | --- |
| **Nov 8, 20:51:30** | `3a0da00` Initial commit | Class starter repo forked (manual). |
| **10:16:37** | `457238c` init expo | `create-expo-app` blank template, SDK 58 (manual). |
| ~10:42 | **Prompt 1 received:** build the app from the TODO file | Claude reads the TODO, `AGENTS.md`, and the project layout. |
| ~10:43 | Cloned the reference repo | Read its README, services, hooks, and Redux store to copy proven SDK 58 patterns. |
| ~10:44 | Fetched Expo SDK 58 docs | audio, filesystem, securestore, document-picker, sharing, router install, jest (per `AGENTS.md`: don't trust training data). |
| ~10:45 | Installed dependencies with `npx expo install` | Router, audio, file-system, secure-store, document-picker, sharing, dev-client, Redux Toolkit, jest-expo. **Hit:** ERESOLVE on react-dom → installed `react-dom` first, then retried. |
| ~10:46 | Installed native AI libraries | `expo-speech-recognition`, `whisper.rn@0.7.4`, `llama.rn`. npm picked `0.13.0-rc.10` → **pinned 0.12.9** (last stable). This choice broke the dev build later (see 11:32). |
| ~10:47 | **Model selection research** | Pulled OpenRouter's live model list (structured-output models plus prices). HEAD-checked Hugging Face URLs for the Whisper and GGUF models; Qwen3-1.7B URL returned 404 → dropped. |
| ~10:49 | Generated synthetic interview audio | `scripts/generate-sample-audio.py` uses macOS `say`, two voices, 16 kHz mono WAV, exact segment timestamps. Outputs: 80.6 s and 67.5 s interviews. |
| ~10:50 | Data models written | `src/models/types.ts` |
| ~10:51–10:55 | Privacy and analysis services | patterns, span merging/propagation, placeholders, LLM detector, payload builder, response validator, OpenRouter client |
| ~10:55–10:59 | Storage, runtime, transcription, model manager, Redux store | **Bug found:** expo-file-system `File` objects change their path after `move()` → rewrote the vault to create fresh `File` objects each time. |
| ~11:00 | First `tsc` | Only failure: whisper.rn types (package `exports` has no `"."`) → fixed with a tsconfig `paths` mapping. |
| ~11:01 | First `jest` run | **All 4 suites failed:** immer ESM not transformed, and the fixtures file was picked up as a test suite. Fixed `transformIgnorePatterns` and moved fixtures out of `__tests__`. |
| ~11:02 | Second `jest` run | 32/35 passed. Real bugs: "**My** son Leo" (capital M) not detected; phone `(313) …` lost its "(" during trimming. Third failure was a bug in the test itself. Fixed → **35/35**. |
| ~11:03–11:11 | Built all screens | Root layout, projects, project, codebook, new session, session hub, record, transcript, privacy review, analysis, export, settings, about. **Hit:** `beforeRemove` listener typing → switched to SDK 58 `usePreventRemove`. |
| ~11:11 | `expo lint` | 13 errors (unescaped apostrophes, two React hooks rules) → fixed. Lint clean, tsc clean. |
| ~11:12 | `expo-doctor` 20/20 checks; `expo export --platform ios` | **Bundle failed:** `buffer` module missing (needed by whisper.rn) → `npx expo install buffer` → bundle OK (3.1 MB). |
| ~11:13 | Expo Go launched on iPhone 17 simulator | Home screen rendered with the demo banner (`shot1.png`). |
| ~11:14–11:16 | Drove the demo via deep links plus a temporary seed route | **Hit:** the first seed route dispatched during render (React error) → moved it into `useEffect`. Verified the session screen (audio 1:20) and privacy review (8 identifiers masked). Found that times displayed in UTC → added `formatLocal`. Seed route deleted afterwards. |
| ~11:17 | README written | Covers models, runtimes, privacy, and tests. |
| ~11:19 | **Prompt 2:** "run the demo for me" | No Simulator.app in this Xcode install → ran headless. Started Metro for Expo Go and gave the phone URL `exp://35.3.193.70:8081` (`demo1.png` at ~11:20). |
| ~11:21 | **Prompt 3:** document, commit, push | Wrote `docs/STATUS.md`. **Hit:** a sed edit failed on the README (`#` delimiter clash) → used the Edit tool instead. |
| **11:22:14** | `82cc38d` first app commit, pushed to `origin/main` | 69 files, about 22k lines including lockfile and audio. |
| ~11:31 | **Prompt 4:** "run the build" | No physical device and no signing identity → targeted the iPhone 17 simulator. |
| **11:32:22** | Dev build attempt 1 started (`npx expo run:ios`) | |
| **11:32:33** | **Build 1 FAILED** (xcodebuild exit 65) | `llama.rn 0.12.9` `RNLlama.mm` uses `RCTCxxBridge`, which React Native 0.88 removed. |
| ~11:33 | Investigated llama.rn release candidates | Unpacked rc.7–rc.10: none use `RCTCxxBridge`. rc.10 was published that day → chose **rc.9** (published Oct 30). tsc and jest re-checked: 35/35. |
| **11:33:46** | Dev build attempt 2 started (fresh `ios/`, pod install) | |
| ~11:35 | **Prompt 5:** "is it done yet?" | Status: about 358 native files compiled, no errors. |
| ~11:37 | **Build 2 succeeded**: 0 errors, 2 warnings; installed and opened | iOS showed an "Open in Interview Lens?" prompt over Expo Go (can't be tapped headlessly). |
| **11:38:37** | `e752d8e` Delete CLAUDE.md | Made by the user on GitHub. |
| ~11:38 | Launched the dev build directly | **Hit:** ATS blocked loading JS from the public LAN IP (35.3.x.x) → connected via `127.0.0.1`. |
| ~11:39 | Dev build verified in full mode | No demo banner; Settings shows "Development build" (`dev3.png`, `dev4.png`). |
| ~11:40 | **Prompt 6:** commit, push, update .md | Updated README and STATUS. |
| **11:40:54** | `44f34a3` build fix + docs commit | **Push rejected** (remote had the CLAUDE.md deletion) → `git rebase origin/main` → pushed. No force-push. |
| ~11:43 | **Prompt 7:** how to run the build in my own terminal | Gave `run:ios`, `expo start`, and `--go` commands, plus signing notes. |
| ~11:47 | **Prompt 8:** `npm expo start` / `--tunnel` not working, QR invalid | Three causes: `npm` typo; port 8081 held by Claude's background Metro; dev-client QR code isn't readable by Expo Go. Stopped the background servers and installed `@expo/ngrok`. |
| **11:49:58** | Tunnel verified | `npx expo start --go --tunnel` → `*.on.expo.app` host; stopped for the user. |
| ~11:52 | **Prompt 9:** create this process doc | |

---

## 2. AI prompting log

**Tool for every interaction:** Claude Code (Anthropic's CLI coding agent), **model Claude Opus 5.5** (`claude-opus-5-5`). It worked as an agent: it read files, ran shell commands, fetched docs, edited code, ran tests and builds, and took simulator screenshots. No subagents or other models were used.

The repo setup before 10:16 (fork, `create-expo-app`) was done manually, not with this tool. Any prompts used in other tools such as UMGPT or Codex are not recorded here; add them if they apply.

### Prompt 1: build the app

> `'/Users/tai/Downloads/UX_Research_App_TODO.md' build the app according to the md file. choose the models wisely.`
> `here is the professor's demo repo link: https://github.com/UMSI669/Voice-To-Text, reference this.`

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Approach:**
  - Read the spec and `AGENTS.md`.
  - Cloned and studied the reference repo.
  - Fetched the SDK 58 docs instead of relying on memory.
  - Picked models from live data: the OpenRouter catalog with prices, and verified Hugging Face file sizes.
- **Model decisions:**
  - Transcription: Apple on-device speech (forced on-device), or Whisper base.en Q5_1 (60 MB).
  - Masking: Qwen2.5-1.5B-Instruct Q4_K_M (1.1 GB), with 0.5B and Llama-3.2-1B options.
  - Cloud: Claude Sonnet 5.5 via OpenRouter as default, with Haiku, Opus and Gemini options.
- **What worked:**
  - Complete TypeScript app with 13 screens and a service layer.
  - Expo Go demo mode, with native modules loaded lazily only outside Expo Go.
  - Synthetic demo audio.
  - 35 unit tests, including a network payload-boundary test.
  - tsc, lint, and expo-doctor all clean; iOS bundle builds.
  - Demo verified in the simulator.
- **What broke → pivot:**
  - npm ERESOLVE (react-dom) → installed react-dom first.
  - whisper.rn types not resolvable (missing `"."` export) → tsconfig `paths`.
  - Jest couldn't load immer ESM; fixtures ran as a test suite → transform allow-list; moved fixtures.
  - Pattern bugs ("My son Leo", phone parentheses) caught by tests → regex/trim fixes.
  - File `move()` changes the object's path → fresh `File` per operation, so a later save can't delete `data.json`.
  - Navigation `beforeRemove` typing → `usePreventRemove`.
  - Metro bundle missing `buffer` → installed it (as the reference repo does).
  - Couldn't tap in the headless simulator → temporary deep-link seed route; its first version dispatched during render → moved to `useEffect`; deleted afterwards.
  - UTC timestamps shown in the UI → local-time formatter.
  - **Unnoticed at the time:** pinning `llama.rn@0.12.9` "for stability" broke the dev build later.

### Prompt 2: run the demo

> `run the demo for me`

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Result:** Expo Go demo running on the iPhone 17 simulator and confirmed by screenshot.
- **What broke → pivot:** this Xcode install has no Simulator.app, so no window could be shown. Gave the user the Expo Go URL for their phone and suggested `--tunnel` for campus Wi-Fi.

### Prompt 3: document and push

> `document what we have for now, and commit and push everything`

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Result:** created `docs/STATUS.md`, mapping every TODO item to done / implemented-but-unverified / not started. Linked it from the README. Checked that no secrets were staged. Committed `82cc38d` and pushed to `main`, the branch the course instructions say to use.
- **What broke → pivot:** a sed edit failed (`#` in "§1" clashed with the delimiter) → used a direct file edit.

### Prompt 4: run the build

> `run the build`

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Result:** read as the native development build. Built for the simulator because no iPhone or signing identity was available.
- **What broke → pivot:**
  - Build 1 failed: llama.rn 0.12.9 uses `RCTCxxBridge`, removed in RN 0.88.
  - Inspected rc.7–rc.10 source → chose 0.13.0-rc.9. It has the fix, and unlike rc.10 it wasn't brand new.
  - Re-ran tsc and tests → rebuilt → **succeeded**.
  - iOS "Open in…" prompt and an ATS block on the public LAN IP → launched the app directly via `127.0.0.1`.
  - Verified full mode by screenshot.

### Prompt 5: progress check

> `is it done yet?`

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Result:** reported progress from the build log (pods installed, about 358 files compiled, no errors) without guessing completion.

### Prompt 6: commit and docs

> `commit and push and update .md`

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Result:** STATUS and README updated with build results, the pre-release pin, and the tunnel note. Committed `44f34a3`.
- **What broke → pivot:** push rejected because the remote had the user's "Delete CLAUDE.md" commit. Checked that it didn't overlap → rebased → pushed normally (no force).

### Prompt 7: running in my own terminal

> `what command should i use if i wanna run build in my own terminal?`

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Result:** commands for the dev build (`npx expo run:ios`, `npx expo start`), the Expo Go demo (`npm run start:go`), rebuilds, and device signing.
- **Miss:** didn't stop its own background Metro first, which caused the port conflict in Prompt 8.

### Prompt 8: tunnel and QR code broken

> `npm expo start and --tunnel not working. QR code provided not valid`

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Diagnosis:**
  1. `npm` instead of `npx`.
  2. Port 8081 held by Claude's background server.
  3. With `expo-dev-client` installed, the default QR code targets the dev build, which Expo Go rejects.
  4. `@expo/ngrok` missing for `--tunnel`.
- **Fix:** stopped the background servers, installed `@expo/ngrok` (dev dependency), and verified a tunnel URL. Recommended `npx expo start --go --tunnel`, or pressing `s` to switch to Expo Go.

### Prompt 9: this document

> `create a detailed process doc. requirements are like this:` *(pasted: Detailed Timeline … AI Prompting Log: exact prompt, model/tool, result)*

- **Tool/model:** Claude Code · Claude Opus 5.5
- **Result:** this file. The timeline was rebuilt from git commit times, build logs, and file/screenshot timestamps.

---

## 3. Lessons learned

- **Check docs and package sources, not memory.** SDK 58 APIs (`usePreventRemove`, async `File.move`, `useAudioStream`) differ from older Expo, and the docs fetch caught them.
- **"Stable" isn't always compatible.** The newest stable llama.rn didn't support React Native 0.88. Compiling the native build early would have caught this sooner than typecheck or tests could.
- **Tests found real bugs.** Synthetic interviews with known ground-truth identifiers made the masking tests meaningful.
- **Clean up background processes.** A leftover dev server caused the user's port and QR confusion.
- **Expo Go vs dev-client QR codes:** once `expo-dev-client` is installed, use `--go` for Expo Go.

---

## 4. Session 2: running on a physical iPhone (Sunday 2026-11-15)

**Goal:** get the dev build onto a real iPhone so on-device transcription can be demonstrated. Expo Go only runs demo mode.
**Outcome:** dev build installed on an iPhone 13 Pro (free Apple ID, personal team). **On-device transcription and a live cloud analysis both confirmed working by the user.** Presentation materials prepared ([PRESENTATION_SCRIPT.md](PRESENTATION_SCRIPT.md)).
Full step-by-step guide and fixes: [IPHONE_DEV_BUILD.md](IPHONE_DEV_BUILD.md). In-class checklist: [CLASS_QUICKSTART.md](CLASS_QUICKSTART.md).

### Timeline

Exact times come from build-log file timestamps. Steps before the first build aren't timestamped and are listed in order.

| Time | Event | Notes |
| --- | --- | --- |
| before 21:29 | **Prompt 10:** QR code not scannable | Diagnosed the dev-client QR code vs Expo Go, plus VS Code terminal rendering. Claude's attempt to start Metro in the background was blocked by the permission classifier, so the user ran it. |
| before 21:29 | **Prompt 11:** Gateway timeout in Expo Go | First tunnel bundle too slow → reload worked. |
| before 21:29 | **Prompt 12:** "only demo mode" | Expo Go can't load the native AI modules → plan the device build. Checked the Mac: Xcode 27.1, 0 signing identities, no phone connected. |
| before 21:29 | **Prompts 13–15:** run device build, where is signing team, VS Code or Xcode? | Phone found but "unavailable": **Developer Mode disabled**, no certificate. User enabled Developer Mode and set the Personal Team in the Xcode app. |
| **21:29:28** | **Prompt 16:** "ready" → device build 1 started (`npx expo run:ios --device <UDID> --no-bundler`) | Verified first: 1 Apple Development identity, phone "connected", Developer Mode enabled, `DEVELOPMENT_TEAM` set. `--no-bundler` so no stray Metro was left running (lesson from session 1). |
| ~21:33 | **Prompt 17:** "is it done yet?" | About 266 compile steps, auto-signing OK, no errors. |
| ~21:36 | **Prompt 18:** "is it done now?" | Linking done; embedding frameworks. |
| ~21:36 | **Prompts 19–20:** keychain "login" password, keeps prompting after *Always Allow* | Mac login password; one `codesign` call per framework (10 + app). Suggested the `set-key-partition-list` fix. |
| **21:38:23** | Build 1: **Build Succeeded (0 errors, 2 warnings)**, then **install failed: `ApplicationVerificationFailed`** | `codesign -dv` per framework: `ExpoModulesWorklets` and `ReactNativeDependencies` **not signed at all** (prompts never approved). |
| ~21:39 | **Prompts 21–22:** "is it done now?" → diagnosis; user ran `security set-key-partition-list …`, replied "done" | |
| **21:40:22** | Device build 2 started (incremental) | No keychain prompts. |
| **21:40:44** | Build 2: installed ✅, launch **denied** | *"profile has not been explicitly trusted by the user"*. Confirmed with `devicectl device info apps` that the app was installed. |
| ~21:42 | User trusted the developer certificate, ran `npx expo start --tunnel`, opened the app | |
| ~21:46 | **Prompt 23:** "it works, transcription is running on device now!!" | Asked for these docs and a class quick-start. |
| ~21:48 | Wrote `IPHONE_DEV_BUILD.md`, `CLASS_QUICKSTART.md`; updated PROCESS, STATUS, README | |
| ~22:00 | **Prompt 24:** analysis says "No compliant provider… (no data collection / ZDR)" | Queried OpenRouter's public `/models/{id}/endpoints`: every Claude 5.5 endpoint **lacks `temperature`**. With `require_parameters: true` OpenRouter rejected all endpoints ("No endpoints found that can handle the requested parameters"), and the app's regex mislabeled that as a privacy error. `denyDataCollection` is on **by default** but was never the cause. |
| ~22:04 | Fix 1 | Removed `temperature` from the payload. Split the 404 mapping (data policy → privacy; requested parameters → "model doesn't support the request"). +2 tests. |
| ~22:05 | **Prompt 25:** now "API key rejected" | The key was valid (same Keychain key passes `/key`). The app grouped **403** with 401 and dropped OpenRouter's reason. |
| ~22:06 | Fix 2 | 403 → "refused although the key is valid" + OpenRouter's (scrubbed) reason; 401 keeps "API key rejected". +1 test. tsc/lint clean, **38/38** tests. |
| ~22:10 | **Prompt 26:** "it works!" + presentation advice | **First live cloud analysis succeeded** on the phone (user-verified). Recommended: live demo on the phone and a 90 s "what it took" story, not a replay of the debug session. |
| ~22:15 | **Prompt 27:** explain Expo Go limits and the Xcode process again | Two-layer (JS vs native) explanation and each signing/build step. |
| 22:44 | **Prompt 28:** presentation script | Wrote `PRESENTATION_SCRIPT.md`. Flagged that local Qwen masking isn't tested on the device yet, so check the demo line tonight. |
| ~22:50 | **Prompt 29:** "open Settings on where?" | Checked `src/app/index.tsx` and `settings.tsx`: it's the in-app **Settings** button (top right of Projects); the "Development build" pill is in the first section. Script updated. |
| ~22:55 | **Prompt 30:** update this file | |

**Session 2 total:** about 20 minutes of build and debug from the first device build to working transcription (21:29 → ~21:46), about 10 minutes to fix the cloud analysis (~22:00 → ~22:10), then presentation prep until ~22:55.

### AI prompting log (session 2)

All prompts: **Claude Code · Claude Opus 5.5** (`claude-opus-5-5`).

| # | Prompt (verbatim) | Result |
| --- | --- | --- |
| 10 | `the QR code generated is not working. cannot scan, not scanable.` | Three causes and fixes: dev-client QR code vs Expo Go → `--go --tunnel`; VS Code terminal rendering; manual URL entry. |
| 11 | `it worked with --go --tunnel, direct me to chrome, after i chose open with Expo Go, it says a network connection was established successfully, but there is a Gateway timeout…` | Explained the first-bundle timeout. Reload / pre-warm with `curl` / `--clear` / hotspot. **Reload fixed it.** |
| 12 | `it worked after reload, used npx expo start --go. however it is only demo mode, cannot do audio to transcript…` | Explained the Expo Go native-module limit. Free Apple ID device-build steps. Noted EAS needs a paid account. |
| 13 | `phone is plugged in, run the device build` | Didn't build: found Developer Mode disabled and no certificate. Gave the exact fixes. |
| 14 | `where can i set the signing team?` | Click path in Xcode with a layout sketch. |
| 15 | `you mean my vscode or the xcode app` | Clarified: the Xcode app, opened via `open ios/InterviewLens.xcworkspace`. |
| 16 | `ready` | Verified identity, device, Developer Mode and team, then started the build in the background. |
| 17–18 | `is it done yet?` / `is it done now?` | Progress from the build log, no guessing. |
| 19 | `it's asking me for my "login" keychain password, which one is that?` | Mac login password; click Always Allow. |
| 20 | `it's keep prompting me even after i choose always allow` | Explained one prompt per framework; `set-key-partition-list` fix. |
| 21 | `is it done now?` | Build OK, install failed. Found the 2 unsigned frameworks; asked the user to run the keychain fix. |
| 22 | `done` | Rebuilt (about 20 s). Installed; launch blocked → trust the certificate on the phone, start Metro with `--tunnel`. |
| 23 | `it works, transcription is running on device now!! please save all of what is above to md files…` | This section, `IPHONE_DEV_BUILD.md`, `CLASS_QUICKSTART.md`, and STATUS/README updates. |
| 24 | `now i am testing the analyze layer. the api key is accepted, but it says "No compliant provider, no provider for this model satisfies your privacy settings (no data collection / ZDR)", however i do not turn these privacy settgins on` | Found the real cause from OpenRouter's endpoint data (unsupported `temperature` + `require_parameters`). Removed `temperature`, fixed the misleading error mapping, added tests. |
| 25 | `now it says "API key rejected"` | Showed the key was valid. Split 401 from 403 and surfaced OpenRouter's reason. Listed what to check (key credit limit, guardrails, account privacy) and suggested a Gemini cross-check. |
| 26 | `it works! holy cow that is so cool! now can you help me with the small presentation tomorrow? … I want to show them the debug session with you … good idea or should i talk about something else?` | Advice: live demo yes; replay the debug session no, tell it in about 90 s as 4 lessons. Mirroring via QuickTime, backup video, fake data only. |
| 27 | `before that, can you explain to me again what won't work on expo go and why, and what process we do on xcode to make sure i can download the app on my phone?` | Two-layer explanation (JS vs native), table of what breaks in Expo Go (checked against `runtime.ts`), and each Xcode/signing step with why it's needed. |
| 28 | `no, but make me a md file with exactly steps that i need to do and things i need to say tomorrow during class` | `docs/PRESENTATION_SCRIPT.md`: tonight / before-class checklists, timed script with [DO] steps, failure table, Q&A. |
| 29 | `[DO] Open Settings, scroll to where it says Development build … open setting on where?` | Located it in the code (in-app Settings button, top right; Runtime section at the top). Fixed the script step. |
| 30 | `nice, update the process md file` | These rows. |

### Lessons learned (session 2)

- **Check the device before starting a 10-minute build.** `devicectl` showed "Developer Mode: Disabled" and `security find-identity` showed 0 identities. Both would have failed the build late.
- **"Build Succeeded" ≠ installable.** Signing each framework separately means one cancelled keychain prompt leaves unsigned code, and iOS rejects the whole app. `codesign -dv` per framework found it in seconds.
- **Fix the keychain partition list once** before device builds on a new Mac. It removes about 11 password prompts per build.
- **`--no-bundler` for background builds** avoids leaving a Metro server that blocks the user's port 8081 (the session 1 mistake).
- **A misleading error message costs more than a missing one.** Two cloud errors ("no compliant provider", "API key rejected") pointed at the wrong cause because the app hid OpenRouter's own message. Checking the provider's public data, then surfacing the raw reason, fixed both in about 10 minutes.
- **New model generations drop parameters.** Claude 5.5 endpoints don't accept `temperature`. Combined with `require_parameters: true`, one optional parameter can make a model look unavailable.
