# Aayush Filing Helper — extension bundle

Unpacked Chrome extension build (from `aayush-filing/extension/dist`, built 2026-11-03).
No dev/test session baked in — this is a clean build; you sign in normally via the popup.

## One-time setup on a new laptop (recommended — no more re-downloading)

```
git clone https://github.com/Aksh28/aayush-filing-extension-dist.git
```

Then in Chrome: `chrome://extensions` → **Developer mode** → **Load unpacked** →
select the `dist` folder inside the cloned repo (not the repo root).

## Every future update, from then on

```
cd aayush-filing-extension-dist
git pull
```

Then go to `chrome://extensions` and click the circular **reload** icon on
the extension's card. That's it — no re-download, no re-extract, no
"Load unpacked" again. The folder Chrome is already pointed at just got
updated in place.

## Alternative: the zip (one-off download, no git)

If you'd rather not use git on that laptop, `aayush-filing-extension.zip`
below has the same files — but every future fix means repeating the
download/extract/Load-unpacked steps from scratch. The `git clone` + `git
pull` route above avoids that entirely.

**2026-11-03 update (6)** — the 2s pre-delay (below) didn't fix it either, which itself was telling: 2 seconds is way more than any real network call needs, so "racing a slow stale reset" was probably never the right theory. Aksh's own instinct ("some loop, it's filling up twice") pointed the right way instead: the earlier "reassert for 2.5s after filling" fix (update 4) was most likely the actual cause -- it could catch the page's own later, legitimate dropdown refresh mid-flight, force a value into a node about to be discarded, and get wiped when the real replacement landed. Removed that reassert loop entirely; the first successful fill (which only ever happens once a matching option is confirmed present) is trusted on its own. The 2s pre-delay stays as harmless insurance.

**2026-11-03 update (5)** — found the real cause of the City/District miss: it only ever happens from the 2nd attendant onward, never the 1st, and never when done by hand. Clicking Add doesn't just clear the entry area for the next attendant -- it re-triggers the same City/District AJAX reset the next attendant's own State fill depends on, a real network round-trip with no event to wait on. A human naturally pauses here; our script didn't, so it could fill the next attendant's city correctly and then watch it get stomped by that stale reset landing late. Fixed with a ~2s pause after each Add, before starting the next attendant's fill, so it's (almost certainly) never racing that reset. The earlier reassert-after-fill attempt stays in as cheap extra insurance.

**2026-11-03 update (4)** — the checkbox fix below worked live (confirmed). The 2nd attendant's City/District still didn't hold after a single 300ms recheck — so instead of one retry, it now keeps re-asserting the correct option for ~2.5 seconds after filling, re-filling every time something knocks it off, since re-filling an already-correct value is harmless.

**2026-11-03 update (3)** — the attendant rebuild below worked live (confirmed). Two follow-up bugs found and fixed from that same live test:
- Each snapshotted attendant block has its own `ownchecks` checkbox that has to be ticked for that attendant to actually be selected/included — our code never touched it. Now ticked automatically right after each block is confirmed added.
- The 2nd attendant's City/District silently came back empty even though it filled correctly — the entry area's own AJAX-driven City/District reset (same mechanism as State→City) was landing late, after our fill, and wiping it back out. Now re-checked ~300ms after filling and re-filled if something reset it underneath us.

**2026-11-03 update (2)** — rebuilt how additional attendants get added, based on the real mechanism confirmed live: the form has exactly ONE reusable entry area (not N clones). Clicking Add snapshots whatever's currently in it into a read-only summary block, then clears it for the next attendant; the last attendant deliberately stays sitting in the entry area, uncommitted, and submits with the rest of the form. The old code was scanning for a new labeled "Surname" field to confirm Add worked — but the snapshotted blocks have no label text at all, so that could never succeed regardless of whether Add itself worked. Now confirmed by watching the attendant list's block count increase instead.

**2026-11-03 update (1)** — fixed real bugs found during live testing on the government portal, confirmed against the actual portal HTML:
- "Add" button for a second/third attendant was a silent no-op — the button's text really is exactly "Add", but the code was picking the first element anywhere on the page whose text normalized to "add", which can grab the wrong control on a long form with more than one "Add" button. Now scoped to only match within the attendant section.
- Date of Birth (patient and attendant) never filled correctly — the portal's field is a plain text input expecting `DD/MM/YYYY`; our data is ISO `YYYY-MM-DD` — now converted at fill time.
- **City/District never filled (real cause, confirmed from the live DOM)**: it's a `<select>` whose options are repopulated by the portal's own AJAX call (`showcities()`) once State is set. The old code filled State and City/District in the same synchronous pass, before that AJAX call could possibly land, so it was matching against a stale options list. Now State is filled first, then the code polls (re-finding the field each tick, since the AJAX response can replace the `<select>` node entirely) until the real options for that state actually appear, before filling City/District.

57 tests pass, including new ones covering all of the above.

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
