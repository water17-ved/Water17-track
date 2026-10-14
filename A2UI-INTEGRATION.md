# Challenge Room — A2UI + Gemini pilot

## What changed in this first implementation slice

- Adds a Gemini planner panel to the existing **AI & Memory** screen.
- Adds an authenticated Firebase HTTPS Function in `a2ui-backend/`.
- The backend verifies the Firebase ID token, calls Gemini using a server-only secret, validates the generated challenge plan, and constructs a limited A2UI v0.9.1 message stream.
- The frontend renders only a small allowlist of A2UI components (`Text`, `Column`, `Row`, `Card`, `Button`, `Divider`, `List`). No generated HTML or JavaScript is executed.
- “Use this plan” opens the existing JSON preview/import flow. It does **not** write to Firestore automatically.
- Existing manual prompt/paste workflow, Firebase rules, challenge creation code, navigation, styling, local backup flow, and PWA shell are retained. The service-worker cache version is bumped so the changed shell can refresh.

This is an initial, constrained protocol pilot—not yet the complete official A2UI renderer. It intentionally does not support data bindings, generated navigation, arbitrary actions, or writes from generated components. The official A2UI project lists v0.9.1 as the current production protocol and documents maintained renderers; before expanding the component set, evaluate adopting its maintained Lit web-components renderer rather than growing a custom renderer indefinitely.

## Backend setup

Prerequisites: Node.js 20, Firebase CLI, access to a Firebase project with Authentication enabled, and a Gemini API key.

1. From the project folder, run `firebase login` and `firebase use --add`. Select the Firebase project that owns the existing Challenge Room authentication. Do not deploy until you have confirmed the project ID matches the app's `firebaseConfig`.
2. Install backend dependencies:

   ```sh
   cd a2ui-backend
   npm install
   ```

3. Set the allowed frontend origin in the Firebase Functions environment. Create `a2ui-backend/.env.<PROJECT_ID>` (do not commit it) with values like:

   ```env
   A2UI_ALLOWED_ORIGINS=https://YOUR_GITHUB_USERNAME.github.io
   A2UI_GEMINI_MODEL=gemini-2.5-flash
   ```

   Use only the origin, not `/Challenges-main/` or any other path. If the site uses a custom domain, list that origin too, comma-separated.

4. Store the Gemini key as a Firebase Functions secret. From the project root:

   ```sh
   firebase functions:secrets:set GEMINI_API_KEY
   ```

   Paste the key into the interactive prompt. Never put the key in `index.html`, a `.env` committed to Git, or any public repository file.

5. Build and deploy only this function:

   ```sh
   cd a2ui-backend
   npm install
   npm run build
   npm run deploy
   ```

   The deployment target is whichever Firebase project you selected in step 1. This does not deploy Firestore rules.

6. Copy the deployed `generateA2ui` HTTPS function URL.
7. Publish the frontend once with this pilot. After the site is live, open **AI & Memory → Gemini connection setup**, paste the full HTTPS function URL, and choose **Save connection**. The URL is stored in local storage on that device, so changing it later does not require another frontend deployment. Other devices need their own URL setting.
8. Test with a non-critical account and confirm sign-in, generation, preview, and challenge creation.

## Security and operational notes

- The endpoint requires a valid Firebase ID token. The browser does not receive the Gemini API key.
- The endpoint does not read or write Firestore. The existing app performs challenge creation only after the user confirms through its existing flow, and existing Firestore rules remain authoritative.
- Origin checks are not authentication; the ID token is required independently.
- Before broad production use, add Firebase App Check and a persistent per-user/IP rate limiter. The function's `maxInstances` limit is only a cost guard, not a complete rate limiter.
- Generated UI is untrusted input. The frontend rejects unknown protocol versions, message types, components, unexpected surface IDs, and oversized component trees. Only the `use_plan` action is enabled.
- If the backend URL is not configured, the new button explains setup is required and the manual AI workflow remains usable.

## Verification checklist

- [ ] Confirm Firebase project ID matches the existing `index.html` config.
- [ ] Build TypeScript with `npm run build`.
- [ ] Deploy `generateA2ui` only.
- [ ] Set the function URL in `A2UI_ENDPOINT` and publish the frontend.
- [ ] Signed-out request is rejected.
- [ ] Signed-in request returns an A2UI v0.9.1 panel.
- [ ] Invalid/oversized prompts and malformed model output fail safely.
- [ ] “Use this plan” opens the existing preview and creates nothing until the normal confirmation button is used.
- [ ] Existing challenge, backup, Firebase Auth, and PWA flows still work.

## Limitations of this first slice

The current static app uses a constrained in-house A2UI v0.9.1 renderer for a narrow pilot. It is not the complete official A2UI implementation. The next step after this pilot passes tests should be a separate, reviewed integration of the maintained Lit renderer (or another maintained renderer) and then incremental component catalog expansion across existing screens.
