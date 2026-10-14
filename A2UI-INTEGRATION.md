# Challenge Room — Firebase AI Logic + A2UI pilot

## What changed
- Replaced manual backend URL entry with a Firebase AI Logic setup link and connection test.
- Gemini API calls use Firebase AI Logic from the existing Firebase app configuration; no Gemini API key is pasted into or stored by the website.
- AI output is parsed as challenge JSON, validated by the existing challenge parser, and rendered through a constrained A2UI v0.9.1 component allowlist.
- Added an Undo control for the most recent tracked Firestore document mutation in the current open session. Undo is refused if the record changed afterward. This is a one-step undo, not a complete multi-step history and does not reverse unrelated local-only UI state.
- Bumped the service-worker cache version.

## One-time Firebase setup
1. Open Firebase Console for project `llenge`.
2. Go to **AI Services → AI Logic → Get started**.
3. Choose the Gemini Developer API and complete the guided setup.
4. Follow Firebase's production App Check setup. Do not use a debug token in the public production site.
5. Deploy the updated frontend files. You do not need to enter a Gemini API key or backend URL into Challenge Room.
6. Sign in on the hosted site, open **AI & Memory**, and press **Test Gemini connection**.

Official setup guide: https://firebase.google.com/docs/ai-logic/get-started?platform=web

## Notes
- Firebase AI Logic must be enabled in the Firebase Console before generation will work.
- The AI Logic service uses the existing Firebase project/app configuration and Google's managed proxy; it does not require a user-entered Gemini API key.
- AI output is only a proposed plan. The existing preview/import path is used before a challenge is saved.
- The renderer accepts a small safe subset of A2UI v0.9.1. It is a pilot, not the complete official A2UI renderer.
- The Undo feature is session-only and only tracks the latest Firestore write routed through the app's tracked write helpers. It is not a substitute for a backup and cannot undo external changes or local-only state.
