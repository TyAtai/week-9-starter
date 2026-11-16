# Project status — Interview Lens

_Last updated: 2026-11-15_

This file tracks progress against `UX_Research_App_TODO.md`. For setup, models, and privacy design, see the root [README](../README.md).

**Legend:** ✅ done and verified · 🟡 implemented, not yet verified on a physical iPhone · ⬜ not started

## Verification so far

| Check | Result |
| --- | --- |
| `npx tsc --noEmit` | ✅ passes |
| `npx expo lint` | ✅ no problems |
| `npm test` (jest-expo) | ✅ 35/35 tests |
| `npx expo-doctor` | ✅ 20/20 checks |
| `npx expo export --platform ios` (Metro bundle) | ✅ builds |
| Expo Go on iOS simulator (iPhone 17) | ✅ home, demo project, interview + bundled audio playback, privacy review (8 identifiers masked in Sample A) |
| Development build on iOS simulator (`npx expo run:ios`) | ✅ compiles (0 errors), installs, launches in full mode; Settings shows "Development build" |
| Development build on a physical iPhone | ✅ installed on iPhone 13 Pro (free Apple ID personal team, 2026-11-15); runs via `npx expo start --tunnel`. See [IPHONE_DEV_BUILD.md](IPHONE_DEV_BUILD.md) |
| On-device transcription in dev build (physical iPhone) | ✅ audio → transcript runs on device (user-verified 2026-11-15) |
| Local LLM masking (llama.rn) and Settings benchmark on device | ⬜ not exercised yet |
| Real OpenRouter analysis request | ✅ succeeded from the physical iPhone (user-verified 2026-11-15), after removing `temperature` (unsupported by Claude 5.5 endpoints) |

## TODO checklist

### 1. Define the MVP
- ✅ Devices: iPhone 12+ · iOS 17+ · English (US) · ≤60 min recommended, 90 min hard stop
- ✅ Masked categories: people, email, phone, URL, IDs, dates, addresses, organizations, locations, identifying context
- ✅ Outputs: summary, codes, themes, verbatim supporting quotes linked to segment ids

### 2. Scaffold
- ✅ Expo SDK 58 + TypeScript + Expo Router (`src/app/`)
- ✅ Models: projects, sessions, segments, redaction spans, identity map, analysis (`src/models/types.ts`)
- ✅ Replaceable services: transcription engines, redaction pipeline, cloud analysis; native modules lazy-loaded only outside Expo Go (`src/services/runtime.ts`)
- ✅ Expo Go demo: bundled synthetic audio, labeled sample transcripts, editable JS masking, real cloud analysis with the user's key

### 3. Capture & transcribe
- ✅ Audio import (expo-document-picker), playback with tap-to-seek, simulator testing with bundled WAVs
- 🟡 Microphone recording (16 kHz mono WAV, background recording, pause/resume, discard guard)
- ✅ On-device transcription running in the dev build on a physical iPhone (Apple on-device speech / Whisper base.en); quality comparison still pending
- 🟡 Availability checks for iOS version, on-device support, locale, model download, permissions, and WAV format; stops instead of uploading
- ✅ Stable segment ids + timestamps, transcript corrections with original kept, manual speaker labels

### 4. Privacy layer
- ✅ TypeScript patterns + project "Always mask" list + manual phrases
- 🟡 Local Qwen/Llama detection via llama.rn, chunked with entity hand-off and transcript-wide propagation
- 🟡 On-device benchmark in Settings (time, chunks, recall vs ground truth); results on oldest phone pending
- ✅ Stable placeholders applied in code (`[PERSON_01]`), wording outside replacements preserved
- ✅ Review screen: Changes / Original / Cloud-bound, accept/reject per entity, approval bound to text fingerprint
- ✅ Local vault with atomic writes, deletion per interview/project/all, no analytics, scrubbed errors; backup behavior documented

### 5. Cloud analysis
- ✅ OpenRouter direct HTTPS; default model `anthropic/claude-sonnet-5.5` plus curated/custom options
- ✅ Key entered at runtime, stored in expo-secure-store (this device only)
- ✅ Payload limited to masked text, segment ids, generic speaker roles, optional masked context; leak check blocks sending
- ✅ Strict JSON schema, validated response, links to segment ids, non-verbatim quotes flagged
- ✅ Cancel, timeout, auth, credits, rate-limit, offline, privacy-routing, and truncation handling; `data_collection: deny` and optional ZDR
- ✅ Live request against OpenRouter (2026-11-15)

### 6. Research workflow
- ✅ Project/session lists, transcript editing, coding view, theme grouping, researcher edits to AI suggestions
- ✅ Local autosave; explicit masked vs original export (Markdown/JSON via share sheet)
- ✅ Project and session deletion

### 7. Validate & package
- ✅ Unit tests: redaction, placeholder consistency, payload boundary, error mapping, validation, store invariants
- ✅ Expo Go demo checked in simulator and on phone; dev build built for simulator and installed on a physical iPhone
- ⬜ Device measurements: transcription quality, missed identifiers on real speech, time, memory, battery
- ⬜ Device edge cases: denied permissions, missing models, malformed files, offline, interruptions
- ✅ Docs: install, model setup, demo limitations, pseudonymization vs anonymity (README)

## Next steps

1. ~~Install the dev client on a physical iPhone~~ ✅ done 2026-11-15. The free-team signature expires after 7 days; refresh with `npx expo run:ios --device`. In-class steps: [CLASS_QUICKSTART.md](CLASS_QUICKSTART.md).
2. Download Qwen2.5-1.5B, then run the Settings benchmark and record results here.
3. Record a short consented test interview; compare Apple vs Whisper transcription.
4. ~~Send one real analysis~~ ✅ done 2026-11-15.
5. Walk through the edge cases in §7 on device.

## Known limitations

- The Simulator GUI is not installed on the dev Mac; simulator checks were done headless with screenshots.
- Pattern masking is heuristic. It misses names that appear without cues (e.g. "Then Jordan said…"); the local LLM and researcher review cover these.
- Apple file recognition timestamps across multiple final results use an offset heuristic; verify on long recordings.
- Imported non-WAV files work only with Apple speech, not Whisper.
- Android has not been built or tested; on-device transcription and masking are iOS-first.
- **llama.rn is pinned to a pre-release, `0.13.0-rc.9`.** The last stable release (0.12.9) fails to compile on React Native 0.88 because it uses the removed `RCTCxxBridge`. Move to 0.13.0 stable when it ships.
