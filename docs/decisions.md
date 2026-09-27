# talk2me — Locked Decisions

Recorded so later sessions don't re-litigate them. Change only with a new,
dated entry saying why.

## 2026-09-27

- **Purpose:** fluency and conversation practice. **Not** pronunciation
  scoring (per-sound / phoneme assessment) — out of scope for v1; would
  need a paid assessment API + server or an on-device model.
- **Speech input:** browser Web Speech API (`SpeechRecognition` /
  `webkitSpeechRecognition`). No API key, no server — app stays static
  and client-only.
- **Speech output:** browser `speechSynthesis`.
- **Hosting:** GitHub Pages via Actions from `main`; repo is public, so
  never commit keys or personal recordings/transcripts.
- **Primary device:** Android phone, Chrome, installed as a PWA.

## Known limits of that choice (accepted, not bugs)

- Chrome's recognizer sends audio to Google by default; not private, needs
  network.
- It normalizes speech toward likely words and may drop disfluencies
  ("um", "uh") — filler-word metrics must be validated on the real phone
  before being relied on.
- No per-word timestamps; pace/pause metrics are approximations from
  result-event timing.
- iOS home-screen PWAs have unreliable speech recognition support.

## Open (must close before the build prompt)

- Who/what is the conversation partner — scripted dialogues vs. an LLM
  (an LLM reintroduces a key + server).
- Which fluency metrics survive a feasibility spike on the actual phone.
