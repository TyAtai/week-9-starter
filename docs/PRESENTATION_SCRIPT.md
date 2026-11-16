# My presentation part — live demo + "what it took"

**My slot:** after my teammates cover purpose/design/structure and privacy. **Length:** about 5–6 min.
**Format:** steps to *do* are in **[DO]**; words to *say* are in quotes. Change the wording to sound like you.

Tech reference if something breaks: [CLASS_QUICKSTART.md](CLASS_QUICKSTART.md).

---

## Tonight (about 20 min)

- [ ] **Do one full test run** on the phone: record → transcribe → privacy review → analysis. Leave this session saved in the app as the **backup session** (completed analysis included).
- [ ] **Check what gets masked** when you say the fake-interview line from step 2. The local Qwen model hasn't been tested on the phone yet. Without it, pattern masking reliably catches the email and the "my name is…" name, but may miss the employer or city. If something is missed, either download Qwen in Settings → On-device models and retest, or plan to show **"mask extra phrase"** live (that's a good feature to show anyway), and adjust the step 4 wording.
- [ ] **Record a backup video** of that run: iPhone Control Center → **Screen Recording**. Save it to the Mac.
- [ ] **Try the projector mirroring:** plug the iPhone into the Mac → **QuickTime Player** → File → **New Movie Recording** → click the **⌄** next to the record button → choose **my iPhone** as Camera. The phone screen appears on the Mac.
- [ ] Charge the phone and Mac. Pack the **USB cable**.
- [ ] Phone: **Do Not Disturb on**, so notifications don't appear on the projector.
- [ ] Check the app still opens. The signature expires around Nov 22.

## 10 minutes before class

- [ ] **[DO]** VS Code terminal, in the project folder:
  ```bash
  npx expo start --tunnel
  ```
- [ ] **[DO]** Open **Interview Lens** on the phone (not Expo Go) → tap the server. If you get a Gateway timeout, wait for `iOS Bundled` and reload. Do this now, not in front of the class.
- [ ] **[DO]** Plug the phone into the Mac, open the QuickTime mirroring window, and make it full screen on the projector.
- [ ] **[DO]** In the app, go back to the **Projects** home screen. Close Settings so the API key isn't on screen.
- [ ] Phone stays plugged in, unlocked, with auto-lock off (Settings → Display & Brightness → Auto-Lock → Never, for today).

---

## The script

### 0. Hand-off (10 s)

> "Thanks. You've heard *why* we built this and *how* the privacy layer works. Now I'll show it actually running, on my own iPhone, with the AI running on the phone itself."

### 1. Prove it's the real build (20 s)

**[DO]** On the app's home screen (**Projects**), tap **Settings** in the **top-right corner** of the header. The first section, **Runtime**, has a green **Development build** label at the top. No scrolling needed. Point to it. *(The OpenRouter API key section is right below. The app only shows the key partly hidden, but don't scroll down or tap into it.)*

> "First: this isn't Expo Go. Expo Go can run our JavaScript, but not our on-device AI engines, because Whisper and Llama are compiled C++ code that has to be built *into* the app. So we built our own version of the app and installed it directly from Xcode. That's what 'Development build' means here."

### 2. Record a fake interview (40 s)

**[DO]** Go back → open the project → **new interview** → **Audio** → **Record**. Say clearly, about 15 seconds:

> *"Hi, my name is Jordan Lee, I'm a nurse at Michigan Medicine in Ann Arbor. You can email me at jordan.lee@example.com. Honestly, the scheduling app we use is so frustrating, I never know when my shift changes."*

**[DO]** Stop the recording.

> "That's a fake interview, but notice it's full of identifying details: a name, an employer, a city, an email."

### 3. Transcribe on the device (40 s)

**[DO]** Tap **Transcribe on device**. While it runs:

> "This transcription is happening entirely on the phone. The audio never leaves the device. For researchers that matters: participants are often promised their recordings stay private, and uploading raw audio to a cloud service would break that promise."

**[DO]** Open the **Transcript**.

> "And here's the transcript, with timestamps. Tap a timestamp and it plays that part of the audio."

**[DO]** Tap one timestamp.

### 4. Privacy review (45 s)

**[DO]** Open **Privacy review** → run detection → show **Changes**, then **Cloud-bound**.

> "[Teammate's name] explained how the masking works; here it is live. The name became a placeholder like PERSON_01, the email, the city and the employer are masked too. This 'Cloud-bound' view is *exactly* what would be sent to the AI. The researcher reviews it and has to approve it. Nothing goes to the cloud automatically."

**[DO]** Approve.

### 5. Analysis (45 s)

**[DO]** Open **Analysis** → show the payload preview → **Send**. *(If it takes more than about 20 s, or fails, say the fallback line below and open the **backup session**.)*

> "Only the masked text goes to the cloud model. It comes back with a summary, codes and themes, and every quote links back to the exact line in the transcript, so the researcher can check the AI didn't make anything up. The researcher can accept, edit or reject each code."

**Fallback line:** *"Campus Wi-Fi is being slow, so here's one I ran earlier"* → open the backup session's analysis.

### 6. What it took to get here (90 s)

**[DO]** Stay on the app screen, or switch to the Mac.

> "Quick story about what it took to get this onto a real phone, because it was most of our debugging time, and we did it working with Claude Code, an AI coding agent."

> "**One: Expo Go isn't enough.** It worked in Expo Go, but only in demo mode, because Expo Go only includes Expo's own native modules, and our AI engines aren't among them."

> "**Two: Apple controls what runs on an iPhone.** To install our own build we had to add an Apple ID to Xcode, turn on Developer Mode on the phone, pick a signing team, and trust the developer certificate on the phone. Every piece of code has to be signed. Our build actually *succeeded*, but iOS still refused to install it. The agent checked each of the 10 frameworks and found two were unsigned, because some password prompts had been cancelled. Fixing the keychain setting and rebuilding solved it in 20 seconds."

> "**Three: error messages can lie.** The analysis first said 'no compliant provider', which sounded like a privacy setting. The real cause was a `temperature` parameter that the newest Claude models don't support. Then it said 'API key rejected', but the key was fine; it was a different kind of refusal. The fix in both cases was to stop guessing and show the real error."

> "**What we learned about working with an AI agent:** it worked best when it checked things before acting. It verified the phone and signing setup before starting a 10-minute build, read the logs, and checked signatures instead of guessing. We logged every prompt in our PROCESS doc."

### 7. Close (10 s)

> "So: on-device transcription, on-device masking, and only reviewed, masked text ever reaches the cloud. Thanks! Happy to take questions."

---

## If something goes wrong live

| Problem | Say | Do |
| --- | --- | --- |
| App won't connect / timeout | "Live demos, right? One second." | Shake → **Reload**. If it fails twice → play the **backup video**. |
| Transcription fails or is slow | "Let me show one I recorded earlier." | Open the backup session's transcript. |
| Analysis slow / error | "Campus Wi-Fi is slow, here's one from earlier." | Open the backup session's analysis. |
| Mirroring fails | "I'll walk around with the phone." / play the video | Hold the phone up, or play the backup video. |
| Notification pops up | — | Ignore it. Do Not Disturb should prevent this. |

**Never show on screen:** the API key field in Settings, real interview recordings.

---

## Likely questions, and short answers

**"Why not just use Expo Go?"**
> "Expo Go only contains Expo's built-in native code. Whisper and Llama are C++ libraries that must be compiled into the app, so we need our own development build."

**"Can we install it on our phones?"**
> "Not yet. With a free Apple account the app only runs on my phone and expires after 7 days. Sharing it needs the paid Apple Developer Program, for TestFlight."

**"Is it really anonymous?"**
> "It's *pseudonymization*, not anonymity. We replace identifiers with placeholders, but context can still identify someone, which is why the researcher reviews every change before anything is sent."

**"What models do you use?"**
> "On the phone: Apple's on-device speech recognition or Whisper base.en for transcription, and Qwen 2.5 1.5B for detecting identifiers. In the cloud, through OpenRouter: Claude Sonnet 5.5 by default, using the researcher's own API key."

**"What does the cloud see?"**
> "Only masked text, segment IDs and generic speaker roles. Never the audio, the original transcript, names, or file names. The app even blocks sending if an identifier slipped through."

**"How much did the AI agent do?"**
> "It wrote most of the code and did most of the debugging, but we made the decisions: what to build, what to trust, and testing it on the real device. All prompts are logged in our PROCESS doc."

**"How long does the build take?"**
> "The first native build was about 10 minutes. After that, code changes reload instantly, because only the JavaScript changes."
