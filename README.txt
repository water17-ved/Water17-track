Challenge Room — App Check / Firebase AI Logic fix

What changed
- Added Firebase App Check initialization using the reCAPTCHA Enterprise site key supplied for this project.
- Initialized App Check before Firebase AI Logic.
- Fixed an extra closing brace after firebaseConfig that would prevent the module script from parsing.
- Kept the existing Gemini model (gemini-2.5-flash). The reported 401 is an invalid App Check token; changing the model name alone will not fix it.

Before testing the hosted site
1. In Firebase Console for project `llenge`, open App Check and confirm that reCAPTCHA Enterprise is registered for the web app with the same site key.
2. Confirm the exact deployed domain is registered/allowed for that reCAPTCHA Enterprise key.
3. Confirm App Check is enabled for Firebase AI Logic for this project. Wait briefly after changing settings, then reload the hosted site and test Gemini.
4. If your key is actually a standard reCAPTCHA v3 key rather than an Enterprise key, this provider must be changed to ReCaptchaV3Provider and App Check configured to match. The provider and key type must match.

The HTML uses Firebase JS SDK 12.10.0 from gstatic. No Gemini API key is embedded in the page.
