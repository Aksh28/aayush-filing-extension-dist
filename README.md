# Aayush Filing Helper — extension bundle

Unpacked Chrome extension build (from `aayush-filing/extension/dist`, built 2026-11-03).
No dev/test session baked in — this is a clean build; you sign in normally via the popup.

**2026-11-03 update** — fixed three real bugs found during live testing on the government portal:
- "Add" button for a second/third attendant was a silent no-op — the button's text really is exactly "Add", but the code was picking the first element anywhere on the page whose text normalized to "add", which can grab the wrong control on a long form with more than one "Add" button. Now scoped to only match within the attendant section.
- Date of Birth (patient and attendant) never filled — the portal expects `DD/MM/YYYY`, our data is ISO `YYYY-MM-DD` — now converted at fill time.
- City/District never filled — the real label has spaces around the slash ("City / District") which didn't match our canonical "City/District" — label matching now tolerates that.

State fills itself automatically once City/District is set (confirmed live) — nothing to fix there.

## Install on the other laptop

1. Download `aayush-filing-extension.zip` from this repo and extract it to its own folder.
2. Open Chrome → `chrome://extensions`.
3. Turn on **Developer mode** (top-right toggle).
4. Click **Load unpacked** → select the extracted folder (the one containing `manifest.json`).
5. Pin the extension, open the popup, sign in with your Aayush Filing account.
6. Open the portal (`https://indianfrro.gov.in/...`) in a tab — the content script only runs there.

## Notes

- `host_permissions` only cover the FRRO portal and this project's Supabase project — nothing else.
- Only the public Supabase anon key is embedded in the bundle (RLS-protected, meant to be public client-side) — no secrets.
- CAPTCHA and the final Submit click are intentionally NOT automated — do those yourself.
