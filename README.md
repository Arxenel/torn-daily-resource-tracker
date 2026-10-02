[README.md](https://github.com/user-attachments/files/32956139/README.md)
# TORN Daily Resource Tracker

One small window that remembers your Torn day for you.

Collapsed, it answers "what have I used today?". Expanded, it answers "where did I use it?". Settings answer "what do I personally want to accomplish today?". The daily reset always follows Torn City Time (00:00 TCT), never your computer clock.

Version 1.0.0. License: MIT. Author: Arxenel [4390395].

---

## Installation

**Tampermonkey or Violentmonkey (desktop browsers, Kiwi on Android):** install the extension, then open the script's GreasyFork page and press Install. The script runs on `https://www.torn.com/*`.

**TornPDA:** add it as a userscript in TornPDA's userscript manager. TornPDA fills in its own API key automatically, so no key setup is needed. TornPDA has no userscript menu, so on touch devices the Hide button is turned off (use Minimize instead).

**Greasemonkey 4:** works through a localStorage fallback, without menu commands. Tampermonkey or Violentmonkey is recommended.

## API key setup

1. Open the tracker, press the gear button, and paste a 16-character key under **API key**, then **Save and test**.
2. The tracker calls Torn's `key/info` endpoint and shows which features your key unlocks.
3. Create a **Full Access** key on Torn at Settings > API Keys (`https://www.torn.com/preferences.php#tab=api`).

**A Full Access key is needed to use every feature, especially energy and nerve used today.** Torn only records energy and nerve spending in your activity log, and only shares that log with Full Access keys (or custom keys that include `log`). A Limited Access key still works for everything else; the energy and nerve rows then show "Full key" and say what they need. The tracker never asks Torn for more than your key allows.

### What each access level unlocks

| Feature | Minimal | Limited | Full | Custom key needs |
|---|---|---|---|---|
| Refills, cooldowns, current bars, travel, organized crime | yes | yes | yes | `bars`, `cooldowns`, `refills`, `travel`, `organizedcrime` |
| Xanax and City Shop counts | yes | yes | yes | `personalstats` |
| Casino token balance and spending | no | yes | yes | `casino` |
| Energy and nerve used today, with breakdown | no | no | yes | `log` |

`timestamp` and `key/info` work with every key.

### Terms of service (Torn API ToS table)

| Data storage | Data sharing | Purpose of use | Key storage and sharing | Key access level |
|---|---|---|---|---|
| Only locally | Nobody | Personal daily tracking of your own resources | Stored locally / Not shared | Full Access (needed for energy and nerve usage; Limited Access works for everything else) |

## Using the window

* **Click** the small window to expand it. **Drag** it (or the header of the expanded panel) to move it. The position is saved.
* Header buttons: refresh now (at most once every 10 seconds), settings, minimize, hide.
* **Hidden?** Use your userscript manager menu: *Show tracker*. On a keyboard, **Alt+Shift+D** toggles it.
* Menu commands: Show tracker, Hide tracker, Expand tracker, Open settings, Reset position.
* The thin line along the top edge shows how much of the Torn day has passed.
* Size: 80% to 120%. Position presets: top left, top right, bottom left, bottom right, or wherever you drag it.
* Colors follow Torn's own light or dark mode (Torn's `dark-mode` class), not your system theme.

### Confidence marks

Every number says how sure it is. Marks can be hidden in settings.

| Mark | Meaning |
|---|---|
| ✓ | Verified from Torn data |
| ~ | Inferred from Torn data that cannot be fully confirmed |
| ≈ | Estimated, may be incomplete (often shown as "≥ n" or "n-m") |
| ? | Unknown |
| STALE | Not refreshed recently |

When something cannot be known, the tracker shows a dash, a range or "≥", never a made-up 0.

## Daily goals

Energy 500, Nerve 100, Xanax 3, City Shop 100, Casino 75 by default. Every goal is editable and can be turned **Off** (then only actual usage is shown).

Goals are personal targets. They are not Torn limits and not your energy maximum. Usage keeps counting after a goal is reached, so 650 / 500 and 4 / 3 are normal. Your current energy bar is shown separately, for example "Current energy 820 (bar maximum 150)".

## Daily reset (TCT)

* Torn City Time is UTC. The tracker reads Torn's server time from the API (`timestamp`) and keeps a corrected clock. Until the first server reply it uses your computer clock in UTC, and it doesn't write anything to the daily ledger until Torn's clock is known.
* At exactly 00:00:00 TCT the daily ledger switches to the new date in every open tab. Nothing needs to be reset by hand.
* The previous day's totals are archived (90 days kept), never deleted.
* Game-state values such as refills and the casino balance are **not** assumed at New Day. They show "Checking after New Day" until fresh Torn data arrives. The tracker refreshes 3 seconds and 35 seconds after New Day.
* After New Day it also re-reads the last minutes of yesterday's activity log, so late entries still land in yesterday's archive.
* Sleep, clock changes and time-zone changes are detected. The tracker resyncs with Torn and never moves the ledger backwards.

## What each section tracks

### Energy (Full Access)

The total of energy recorded in today's activity log entries (`energy_used`), split into Gym, Attacks, Revives, Hunting and Other by Torn's log category and title. Gym trains that have no energy value are listed as unknown, not guessed.

### Nerve (Full Access)

Nerve recorded in today's log: crime entries (`nerve`) and any entry with `nerve_used`, split into Crimes and Other. Crime entries without a nerve value are counted as unknown, and the total is then shown as a minimum.

### Xanax

Xanax taken today comes from your `xantaken` personal stat compared with its value at 00:00 TCT. That starting value is proven by, in order:

1. The stat's own "last updated" time showing no change since New Day.
2. An observation after New Day of a stat that hadn't changed since before it.
3. Matching observations on both sides of New Day.
4. Torn's historical personal stats (marked ~).

If none of these is available, you see a range or a minimum. With Full Access, Xanax-use log entries confirm the number, but only when they fall inside what the counter already proves. Bought, received or sent Xanax is never counted.

The drug cooldown is shown next to it.

### City Shop

Items bought today from Torn's East Side shops, compared with Torn's allowance of 100 items per Torn day. This uses the `cityitemsbought` stat (same baseline method as Xanax). With Full Access it is confirmed by the quantity in your "Item shop buy" log entries. Failed purchases don't change either source. The tracker never buys anything.

### Casino tokens (Limited Access)

* Shows the daily base (75), your live balance from `/user/casino`, tokens spent today and any other tokens gained.
* Torn has no "tokens spent" field, so spending is measured from drops in your balance while the tracker is watching.
* If it watched continuously since New Day, the number is marked ~. Otherwise it is shown as a minimum (≥), because play while no Torn tab was open can't be seen.
* Only casino tokens are tracked, not casino points.

### Refills

Energy, nerve and casino refill status straight from Torn (`refills`). Special refills are shown when you have any. The tracker never uses a refill.

### Cooldowns

Drug, booster and medical cooldowns. Each one is stored as a target time from Torn's server and ticks locally every second. It is refreshed from Torn on every live update.

### Organized crime (OC 2.0)

Your current crime from `/user/organizedcrime`: name, status, time until ready, or recruiting progress. It also tells you if your faction isn't on Organized Crimes 2.0.

### Travel and OC warning

* **Anywhere in Torn:** using `/user/travel` and your OC ready time, the tracker estimates when you could be back in Torn. It shows **Safe**, **Caution** (back within your safety buffer) or **Conflict** (still away when the OC is ready). OC 2.0 only starts when every member is in Torn and out of hospital.
* **On the Travel page:** it reads the flight data Torn already loaded on that page. It shows Torn's own per-destination OC flag next to the tracker's estimate. If they disagree, both are shown and Torn's check is the one that counts.
* Settings: planned time abroad and safety buffer. Flight times get Torn's documented 3% variance.
* Nothing is clicked, blocked or chosen for you.

## Refresh schedule

| What | How often |
|---|---|
| Live data (bars, cooldowns, refills, travel, OC, casino, server time) | every 60 s (30 s / 2 min selectable) |
| Personal-stat counters | every 2 min |
| Activity log (Full Access only) | every 2 min, only entries since the last read |
| Key permissions | at start, after a key change, every 6 h |
| Historical baseline | only when needed, at most 3 tries per day |

Hidden tabs refresh three times slower (except within 2 minutes of New Day). Countdowns never poll the API.

With several Torn tabs open, only one tab talks to the API. The others show the shared data. The tracker stays under 20 requests per minute; Torn allows 100.

## Privacy

* Your API key is stored locally in this userscript's storage. On TornPDA it is in the app's storage for torn.com.
* The tracker talks only to Torn's official API at `https://api.torn.com`. Your key is never sent anywhere else, never logged and never shown in full.
* No analytics or telemetry are collected. All history stays in your browser.
* On the Travel page it reads the travel data Torn already loaded. It never loads pages in the background.

## Security

* Readable, unminified, single-file source with no external libraries, no remote code, no `eval`, and no iframes.
* Exactly one network call site, and it only targets `https://api.torn.com/v2`.
* Requests are tagged with the comment `TRTracker`, so you can see them in your API key log.
* A key that Torn rejects (or that is paused, inactive or owned by a jailed player) is never used again until you change it or press Try again. This avoids the temporary IP bans Torn applies to repeated invalid-key requests.

## Local data

Storage keys (prefix `trt:`):

| Key | Contents |
|---|---|
| `settings` | Your preferences |
| `apikey` | Your API key |
| `day`, `prev` | Raw observations for today and yesterday |
| `hist` | Daily totals, 90 days |
| `snap` | Last live data |
| `meta` | Key permissions, clock offset |
| `leader`, `rev` | Tab coordination |
| `keyflag` | Rejected-key marker |

Settings > Your data offers:
* **Export data** (JSON, never includes the API key)
* **Import**
* **Reset history**
* **Reset settings**
* **Remove key**

## Troubleshooting

| You see | What to do |
|---|---|
| API key required | Add a key in settings. |
| Torn rejected the API key | Paste a new key, then Save and test, or press Try again. |
| Energy and nerve show "Full key" | Use a Full Access key, or a custom key with the `log` selection. To hide the note on a Limited key, turn off the Energy and Nerve sections. |
| STALE badge | Torn did not answer recently. The tracker retries automatically. |
| "Torn rate limit reached" | Another tool used your 100 requests per minute. The tracker waits 60 s. |
| "…" for Xanax or City right after New Day | Normal for up to about 35 s, while the tracker waits for an answer it knows is from after New Day. |
| Window is gone | Userscript menu: Show tracker, or Alt+Shift+D, or menu: Reset position. |
| Travel page says data is not available | Torn changed or hasn't loaded the page model. The API-based travel check still works. |

## Known limitations

* Energy and nerve totals need activity log access. They count only log entries that carry an energy or nerve value. Actions whose log entries carry no such value are not included, and flagged entries are shown as unknown.
* Breakdown categories come from matching Torn's log category and title text.
* Casino spending is measured from balance changes seen by the tracker:
  * play while no Torn tab is open is not seen;
  * a spend and a gain inside one refresh interval can cancel out;
  * a token refill makes that interval ambiguous.
  The tracker marks all of these as estimates.
* Torn's historical personal stats are documented only loosely ("converted to nearest date"). They are checked against the returned timestamp and always marked ~.
* The City Shop counter assumes `cityitemsbought` counts items. With Full Access this is cross-checked against purchase log quantities.
* Torn doesn't document the Travel page model (`#travel-root` data). If Torn changes it, that part shows a message instead of guessing. Delays added by another player's Detective Agency watchlist can't be predicted.
* If the browser was closed for more than a whole Torn day, that older day keeps whatever was captured before it closed.
* Tab handover after a tab freezes can take up to about 30 seconds.
* Torn's activity log read limit (50,000 rows per day per category) pauses log reading until the next New Day if reached.
* Not yet run against the live Torn API with a real key. Before publishing, run the checklist in COMPLIANCE_AUDIT.md.

## Credits and reuse

TORN Daily Resource Tracker was created by **Arxenel [4390395]**.

If you wish to recreate or modify this script, please don't forget to mention me, Arxenel [4390395], as the original author. The script is MIT licensed: you are free to change and share it, as long as every copy or modified version keeps the copyright notice crediting Arxenel [4390395].

## Torn compliance

This userscript is designed as a read-only information tool.

It uses Torn API data and, where applicable, data from Torn pages the player is actively viewing.

It does not automate attacks, travel, purchases, item use, refills, Casino actions, CAPTCHA interaction, or other gameplay.

Users should review Torn's current rules and API terms before use.

Designed around Torn's published API and scripting rules.

---

Like the script? Send a Xanax to [Arxenel [4390395]](https://www.torn.com/profiles.php?XID=4390395).
