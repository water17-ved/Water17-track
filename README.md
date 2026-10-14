# Challenge Room — Google-only build

- Android/browser back navigation: the device back button returns through the previous in-app screen or challenge pane instead of immediately leaving the site. The top-left Back control follows the same navigation history.

## Included fixes
- Added a **Log out** button under Profile → Settings.
- Waits for Firebase's authentication state on page load before showing either the app or the login screen, preventing the login page from flashing on reload for a signed-in user.
- Keeps Google as the only sign-in method and uses Firebase local persistence.
- Improved profile-picture rendering so photos are consistently circular and cropped, and broken images fall back gracefully to initials. The picker explicitly accepts common image extensions (JPG/JPEG, PNG, WebP, GIF, BMP, AVIF, SVG, TIFF, HEIC/HEIF); browser-decodable formats are resized and converted to JPEG before saving. HEIC/HEIF support depends on the browser decoder, so convert those to JPG/PNG/WebP if they cannot be decoded.
- Updated the service-worker cache version so deployed browsers fetch the new files.

## Deploy
1. Extract this ZIP.
2. Upload/replace `index.html`, `sw.js`, `manifest.json`, and `icon.svg` in the root of the `water17-ved/Challenges` GitHub repository.
3. Wait for GitHub Pages to redeploy, then refresh the site. If an old version persists, close the site and reopen it.

Firebase Authentication must have Google enabled and `water17-ved.github.io` listed under Authentication → Settings → Authorized domains.

## Profile picture note
JPG/JPEG, PNG, and WebP are the safest formats. GIF, BMP, AVIF, SVG, and TIFF may also work if the browser can decode them; uploaded photos are normalized to JPEG. HEIC/HEIF acceptance is enabled in the picker, but decoding depends on the browser and may require conversion to JPG, PNG, or WebP.

## Validation
The JavaScript module passed `node --check`. Firebase sign-in, Firestore writes, and the live GitHub Pages deployment were not tested from this environment.

## Challenge tab upgrades
- The Challenge tab now opens with two primary choices: **Challenge Market** and **My Challenges**.
- **Challenge Market** lists live, discoverable challenges that the signed-in user has not joined and provides a Join action.
- The challenge builder supports a nested study tree: **Subject → Chapter → Topic → Task**. Tasks keep their subject/chapter/topic metadata and render grouped into that tree. Each task can be daily or one-time.
- **My Challenges** has separate actions for **Create challenge** and **Create mega challenge**.
- Standard challenge creation only builds its own Subject → Chapter → Topic → Task tree.
- The separate **Create mega challenge** screen lets the creator choose existing standard challenges, set a title/description/date range, and combine them into one parent mission. Component challenges remain independent and appear as branches in the mega challenge. Mega challenges are excluded from the component picker to avoid accidental recursive nesting.
- Newly created challenges are marked `discoverable: true` for market listing.

## Firestore market permission
The market uses a Firestore query for documents where `discoverable == true`. If the Market says it is unavailable, merge an appropriate list permission into your existing `/challenges/{challengeId}` Firestore rules (do not replace your whole rules file blindly):

```text
allow list: if request.auth != null && resource.data.discoverable == true;
```

Keep your existing ownership, membership, and write permissions intact. Firestore rules are not included in this ZIP.


## Editing challenges and task frequency

Challenge owners can open a challenge and choose **Edit challenge** to update its title, description, start/end dates, and (for standard challenges) task text, subject/field, chapter, topic, and frequency. Tasks can be added or removed before saving. Each task can be marked **Daily** or **One-time**. Mega challenge owners can edit the mega challenge details and add/remove linked standard-challenge branches. Changes are saved to Firestore and are visible to other members.

## AI Plan and Memory backups
- The new **AI** tab includes a starting prompt for planning in a separate AI conversation and a final prompt that requests strict JSON.
- Paste the AI's final JSON into **AI response**, preview the parsed task tree, then create the challenge. The AI itself is not connected to the site; the user copies prompts and pastes the response back.
- **Memory & local backups** exports a `.json` file containing local profile settings, joined challenge references, accessible challenge definitions, and the current member's progress where Firestore rules permit reading it. Import restores profile settings and recreates challenges owned by the original account as new copies with new codes.
- A backup file is local to the user. Keep a copy somewhere safe; it is not automatically synced to cloud storage.

## Guest access
- **Continue as guest** uses Firebase Anonymous Authentication. Enable it in Firebase Console → Authentication → Sign-in method → Anonymous. Google sign-in can later be linked to the guest account to preserve ownership where Firebase permits linking.
- Guest sessions are tied to the browser's Firebase persistence. Clearing browser data or losing the guest session can make cloud challenges difficult to recover; export a memory backup and link Google for safer long-term access.

## Installability / PWA
- The manifest now includes 192×192 and 512×512 PNG icons in addition to the SVG icon, and the service worker caches the app shell and icons.
- The Profile → Settings → **Install** action invokes Chrome's install prompt when the browser offers it. If no prompt appears, open the HTTPS GitHub Pages URL directly in Chrome, refresh after deployment, and use Chrome ⋮ → **Install app** or **Add to Home screen**. Some browsers only offer Add to Home Screen and do not expose the install prompt event.
- The app must be served over HTTPS (GitHub Pages is HTTPS). A local `file://` copy is not installable as a PWA. If Chrome still does not offer installation, inspect the page's manifest/service-worker status in Chrome DevTools → Application; check for failed manifest/icon requests and ensure the service worker has activated.

## Temporary friend progress sharing (10-minute access)
- In a challenge's **Board** pane, tap **Request progress** beside a friend. The friend can accept or decline from **Profile → Progress access requests**.
- On acceptance, a snapshot of that one challenge's tasks, completion data, points, and journal notes is stored in `temporaryProgressShares` for 10 minutes. The requester can view it from their Profile panel while it remains valid. Expired snapshots are rejected by the app and clients attempt to delete expired documents when the panel is refreshed.
- Requests are stored in `progressAccessRequests`; requests expire after 24 hours if unanswered.
- **Firebase note:** client-side cleanup is best-effort; it cannot guarantee server deletion at exactly 10 minutes if nobody opens the app again. For automatic server cleanup, configure Firestore TTL on the numeric `expiresAt` field after changing it to a Firestore Timestamp, or use a scheduled Cloud Function. TTL deletion is asynchronous, so the app still enforces the expiration timestamp before showing data. If using numeric milliseconds as in this build, Firestore TTL will not work until `expiresAt` is stored as a Timestamp.
- Firestore rules must explicitly restrict access to these collections so only the requester and target can read/write the matching request, and only the approved viewer and owner can read a temporary share. Do not use open/public rules. The app reports a permissions message if your current rules don't permit these operations.
- This feature stores a temporary copy only for the selected challenge; it does not create a permanent friend data archive. Review shared journal notes before approving if you don't want them included.

## Daily Report (Activities)
The Activities view now includes a local-first Daily Study Report for Physics, Chemistry, and Mathematics.
- Add multiple subject sessions; each starts as **Questions** and cycles by tapping the session-type control: Questions → Revision → Notes Making → Questions.
- Each session supports optional start/end times, topic/notes, and manual test score. Questions sessions additionally support questions solved and correct/attempted counts for calculated accuracy.
- Optional per-subject question targets count only Questions sessions. Optional study-hour targets compare against timed work of all session types.
- Drafts are stored in browser localStorage as you type. Use **Save / Update Report** to mark a date as saved in the calendar. **Copy Today's Report** copies a WhatsApp-formatted summary of the selected report to the clipboard; it does not send messages automatically.
- Use the calendar to open saved dates and edit reports. All Daily Report data is stored on the device and is not written to Firebase. Clearing browser/site data can remove it.

## Daily Report validation
The inline JavaScript passes `node --check`. This package has not been tested against the live Firebase project or deployed GitHub Pages site from this environment.

## Mega folders, Home tasks and Mega hub (v18)
- **My Challenges / Home:** a mega challenge is now a folder row. Tap it to expand in place; inside you get **Open mega challenge** plus every branch challenge. Challenges that sit inside a mega are hidden from the top level so they never appear twice. Open/closed state is remembered on the device.
- **My Challenges** shows full detail: dates, target count, and **Daily done/total (today)** and **One-time done/total** for every challenge and for each mega folder as a whole. Home keeps a compact version of the same list.
- **Home → Today's daily tasks:** every live standard challenge lists its daily tasks grouped by challenge (with the parent mega in small text). Tick them right there. Points are calculated exactly as inside the challenge.
- **Home → fourth tile** now enters the mega challenge: opens it directly if you have one, shows the Mega Challenges screen if you have several, or starts "Create mega challenge" if you have none.
- **Challenge tab → Mega Challenges** (new third box): one card per mega with total daily and one-time progress bars, then every challenge inside it with its own daily and one-time done/total.
- The mega challenge page now starts with a combined progress summary, and each branch row shows its counts.
- A challenge can belong to only one mega: challenges already inside another mega are not offered in the create/edit pickers.
- Home reads each joined challenge and your own member document once when it opens. No new Firestore rules are needed because members already read their own member document. Cache version bumped to v18.

## Gemini + A2UI pilot (initial integration)
The AI & Memory screen now contains an optional Gemini-powered A2UI challenge planner. It requires a separately deployed authenticated Firebase HTTPS Function; no Gemini key is placed in the frontend. See [A2UI-INTEGRATION.md](A2UI-INTEGRATION.md) for setup and deployment. Until the backend URL is configured, the original manual prompt/paste workflow remains available. This pilot does not modify Firestore rules and does not save generated plans until the existing user-confirmed challenge creation flow is used.


## Firebase AI Logic + A2UI pilot
See `A2UI-INTEGRATION.md` for one-time Firebase Console setup and testing. No Gemini API key or backend URL needs to be entered into the live site.
