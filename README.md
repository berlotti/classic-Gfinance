# Classic Watchlist

A single-file HTML page that shows your stock watchlists in the style of the **old Google Finance** — ticker badge, company name, price, daily change, and a coloured percentage pill, one row per holding.

**[Open it here](https://berlotti.github.io/classic-Gfinance/)** · **[Source on GitHub](https://github.com/berlotti/classic-Gfinance)**

It reads its data from a Google Sheet that you own. No account, no API key, no server, no build step. Use the hosted page, or download `index.html` and open it from your desktop — both work identically.

---

## Why this exists

Google retired the old Google Finance layout in favour of a card-heavy design built around news, discovery and charts. That is a reasonable product for browsing. It is a worse product for the thing a lot of people actually used the old one for: **glancing at a list of tickers and seeing, in one screen, what moved today.**

The old watchlist put roughly a dozen holdings above the fold in a dense table you could read in two seconds. The replacement spends that space on other things.

This page brings that view back, with three deliberate differences:

- **Your data lives in your spreadsheet**, not in a product that can be redesigned out from under you. If this page ever stops working, your watchlist is still a Google Sheet.
- **It loads once.** No live ticking, no polling, no attention-grabbing. Open it, read it, close it.
- **It is one HTML file.** You can read the whole thing, host it anywhere, or keep it on your desktop forever.

There is no official Google Finance API — it was deprecated in 2011 and shut down in 2012. The only supported way to get Google's finance data is the `GOOGLEFINANCE()` function inside Google Sheets, which is exactly what this setup uses.

---

## Quick start

1. Open **[berlotti.github.io/classic-Gfinance](https://berlotti.github.io/classic-Gfinance/)**, or download `index.html` and open it in your browser.
2. Press **Demo sheet** to see it working with an example spreadsheet.
3. For your own data, follow [Making your own sheet](#making-your-own-sheet) below.

The page remembers the last sheet you used, so from then on it opens straight to your watchlist. A fresh browser shows nothing until you load a sheet.

---

## Making your own sheet

About ten minutes. You need a Google account and nothing else.

### 1. Copy the demo sheet

The fastest route is to copy a sheet that's already laid out correctly:

```
https://docs.google.com/spreadsheets/d/1UvTu-ZDs8euxx4h5qWev9wBgc-MBPdy7ZzE9ePHGEh0/edit?usp=sharing
```

**File → Make a copy.** You now own the copy and can edit it freely. (Prefer to start empty? See [Building a sheet from scratch](#building-a-sheet-from-scratch).)

### 2. Put your own tickers in

In each tab, replace the tickers in **column A**. Every other column is a formula that reads column A, so once a ticker is in place the rest of the row fills itself in. To add a holding, type the ticker in the next empty cell of column A and drag the formulas down from the row above.

Tickers use the `EXCHANGE:TICKER` form Google Finance itself uses — look the stock up on google.com/finance and read it off the page:

| Market | Prefix | Example |
|---|---|---|
| Nasdaq | `NASDAQ:` | `NASDAQ:AAPL` |
| NYSE | `NYSE:` | `NYSE:V` |
| US over-the-counter | `OTCMKTS:` | `OTCMKTS:TCEHY` |
| Amsterdam | `AMS:` | `AMS:ASML` |
| Paris | `EPA:` | `EPA:MC` |
| Frankfurt / Xetra | `ETR:` | `ETR:SAP` |
| London | `LON:` | `LON:SHEL` |
| Milan | `BIT:` | `BIT:ENI` |
| Zurich | `SWX:` | `SWX:NESN` |
| Hong Kong | `HKG:` | `HKG:0700` |
| Tokyo | `TYO:` | `TYO:7203` |
| Toronto | `TSE:` | `TSE:RY` |
| Australia | `ASX:` | `ASX:BHP` |
| Indices | `INDEXSP:` `INDEXDJX:` `INDEXNASDAQ:` `INDEXDB:` | `INDEXSP:.INX` |
| Currencies and crypto | `CURRENCY:` | `CURRENCY:EURUSD`, `CURRENCY:BTCUSD` |

If a ticker returns `#N/A`, Google doesn't recognise that symbol — copy the exact form shown on google.com/finance.

### 3. Name your tabs

Each tab becomes one list in the page. Two rules:

**Name the tabs `1`, `2`, `3`, …** — plain numbers, nothing else. Not `Sheet1`, not `Tech`, not `01`. Right-click a tab → Rename.

**Put the list's name in cell A1**, and leave the rest of row 1 empty. That's what appears on the tab button — "Watchlist", "Pension", whatever you like.

Up to 24 tabs. Gaps are fine: delete tab `2`, keep `1` and `3`, and both still appear.

> **Why numbers?** The page can't ask Google "what tabs exist?", so it asks for tab 1, tab 2 and so on until it stops finding new ones. Asking for a tab that doesn't exist doesn't return an error — Google quietly serves tab 1 again — so numbered tabs are what let the page tell where your lists end.

### 4. Share it

**Share → General access → Anyone with the link → Viewer → Copy link.**

**Viewer**, not Editor. The page only ever reads; edit access gains you nothing and means anyone who gets the URL can rewrite or delete your watchlist, with no useful audit trail. And since anyone with the link can read it, keep the sheet to tickers and prices — share counts, cost prices and account numbers don't belong in a link-shared document.

You do **not** need *File → Publish to web*. Those links look like `/d/e/2PACX-…` and won't work here.

### 5. Load it

Paste the link into the box at the top of the page and press **Load**, or press Enter. A bare sheet ID works too.

Your tab names appear as buttons, with **All** first — a merged view of every list, deduplicated by ticker. Click the title above the box to name the sheet; Google's data endpoint doesn't expose the spreadsheet's own title, so this is a label you set, stored per sheet.

---

## Keeping the numbers fresh

This is the part that surprises people.

`GOOGLEFINANCE` values are **not live**. They are delayed up to 20 minutes, and they only recalculate **while the spreadsheet is open**. A sheet nobody has opened since yesterday will happily serve yesterday's prices.

The distinction that matters:

- **Opening the spreadsheet refreshes it.** The formulas are evaluated on Google's servers, so this works even in read-only mode — your permission level doesn't stop the formula fetching. Loading or refreshing the sheet's browser tab pulls a fresh block of data, and it keeps refreshing in the background roughly every couple of minutes while it stays open.
- **This page reading your sheet does not.** It fetches a data-export endpoint, which serves the values already stored in the cells. Pressing **Refresh** here re-reads the sheet; it cannot make the sheet re-fetch from Google.

So the routine is: **open the sheet, then press Refresh on the page.** Keep the spreadsheet open in a tab while you work and it stays current on its own.

You can see the difference in the data itself: a row will sometimes show a price from 10:32 next to a change from 10:29, because each cell was last evaluated at a different moment. A fresh recalculation would have computed them together. For the same reason, two sheets holding the same stock will rarely agree to the cent — they're snapshots from different moments.

Sharing as Editor does **not** help; permissions and recalculation are unrelated. Edit access matters for one thing only: just editors can change **File → Settings → Calculation → Recalculation**, where "On change and every minute" nudges things along. It still only applies while the sheet is open.

> **Worth knowing:** the "loaded 15:14" in the page's footer is when *the page* read the sheet, not when the sheet last recalculated. If nobody has opened the spreadsheet for a while, the page can load at 15:14 and still show you this morning's prices. When the numbers matter, open the sheet.

---

## Using the page

| Control | What it does |
|---|---|
| **Tabs** | One per list, from your numbered sheet tabs. `←` / `→` also switch between them |
| **All** | A merged view of every list, deduplicated by ticker. Hovering a row shows which lists it came from. Only this tab shows the summary strip |
| **Column headers** | Click to sort by name, price, change or **% Chg**. Click again to reverse. Defaults to biggest movers first |
| **Refresh** | Re-reads the sheet. Otherwise the page loads once and stays put |
| **Theme** | Cycles Auto → Light → Dark. Auto follows your operating system |
| **Row click** | Opens that stock on Google Finance in a new tab |
| **Demo sheet** | Loads the example sheet |

**Greyed-out rows** are on an exchange that isn't trading right now, worked out from the exchange in the ticker and your computer's clock, and re-checked every minute while the page is open. It knows the lunch breaks in Hong Kong, Tokyo, Shanghai and Singapore. It does **not** know public holidays — see the disclaimers.

Your sheet, chosen tab, sort order, theme and sheet name are remembered in your own browser. Nothing is sent anywhere.

---

## Sheet rules, in full

| Rule | Detail |
|---|---|
| Sharing | Anyone with the link → **Viewer** |
| Tab names | Plain numbers: `1`, `2`, `3`, … up to 24. Gaps allowed |
| Row 1 | List name in **A1**, rest of the row empty. Must contain **no numbers** |
| Header row | **Not allowed.** Data starts in row 2 |
| Ticker column | Required. Must be the **leftmost text column** |
| Price column | Required. The **first** numeric column |
| Change column | Optional. The **second** numeric column |
| % change column | Optional. The **third** numeric column |
| Name column | Optional text column, to the right of the ticker |
| Currency column | Optional. Three-letter codes like `USD`, `EUR`, `HKD` |
| Blank / `#N/A` rows | Ignored, so spare formula rows below your holdings are fine |

Two constraints the page can't work around, because it identifies columns by their content:

- **The ticker must be the leftmost text column.** Put a currency or name column to its left and that column gets mistaken for the ticker.
- **Numeric columns are read left to right as price, change, % change.** Supply only two numbers and the second is treated as the change, never the percentage.

If you supply only one of change or % change, the other is calculated for you. Without a currency column, prices render as plain numbers rather than `$` or `€`.

---

## Building a sheet from scratch

1. New spreadsheet. Rename the first tab to `1`.
2. In **A1**, type the name of the list, e.g. `Watchlist`. Leave B1:F1 empty.
3. In **A2**, type your first ticker, e.g. `NASDAQ:AAPL`.
4. In **B2:F2**, paste these five formulas:

```
=GOOGLEFINANCE($A2,"price")
=GOOGLEFINANCE($A2,"change")
=GOOGLEFINANCE($A2,"changepct")
=GOOGLEFINANCE($A2,"name")
=GOOGLEFINANCE($A2,"currency")
```

5. Select B2:F2 and drag down as far as you want rows.
6. Fill column A with your tickers.
7. Add more tabs named `2`, `3`, … the same way.
8. Share as **Viewer** and load the link.

To keep `#N/A` out of the sheet while it recalculates, wrap each formula: `=IFERROR(GOOGLEFINANCE($A2,"price"),"")`.

---

## Troubleshooting

**"Couldn't read the sheet" / "Google returned a web page instead of data"**
The sheet isn't shared publicly. **Share → General access → Anyone with the link → Viewer**. By far the most common cause.

**"Couldn't find a price column in the sheet"**
Usually a header row — remove it, only the A1 title row is allowed. Otherwise the price column contains text rather than numbers; check the cells aren't formatted as plain text.

**Prices, changes and percentages in the wrong columns**
The ticker must be the leftmost text column, and numbers are read left to right as price, change, percentage.

**Only one list shows, or a tab is missing**
Tab names must be plain numbers. `Sheet1`, `Tech`, `1.` and `01` won't be found.

**Two lists collapse into one**
If a tab has exactly the same tickers *and* the same A1 name as tab 1, the page can't tell it apart from Google's silent fallback and hides it. Change the name or the contents.

**Numbers are stale, or two sheets disagree**
Open the spreadsheet in a browser tab and refresh it — that forces a fresh pull, even in read-only mode — then press **Refresh** here. See [Keeping the numbers fresh](#keeping-the-numbers-fresh).

**A row shows `#N/A` or no price**
Google doesn't recognise that ticker. Look it up on google.com/finance and copy the exact `EXCHANGE:TICKER` form.

**Everything is greyed out**
Greying means the exchange isn't trading, based on your computer's clock. Check your system time and time zone. Public holidays aren't known to the page, so a market closed for a holiday will still look open.

**The page opens empty saying "No sheet loaded"**
That's the first-run state. Paste a link or press **Demo sheet**. If it happens after you'd already loaded a sheet, your browser cleared its local storage — paste the link again.

---

## How it works

The page asks Google's spreadsheet visualisation endpoint for each tab as JSON:

```
https://docs.google.com/spreadsheets/d/<SHEET_ID>/gviz/tq?tqx=out:json&headers=0&sheet=1
```

It loads that with a `<script>` tag rather than `fetch()`. This matters: Google doesn't send CORS headers on that endpoint, so a normal `fetch()` from a page opened off your disk is blocked by the browser and fails with *"Failed to fetch"*. Script tags aren't subject to CORS, and the endpoint will wrap its reply in a function call for exactly this purpose.

Consequences worth knowing:

- The only network request the page makes is to `docs.google.com`, for your sheet. No proxies, no CDNs, no analytics, no third parties.
- It works from a `file://` path, a USB stick, or any static web host. The hosted copy is GitHub Pages serving this same file.
- Preferences are stored in a cookie *and* in local storage. Browsers refuse cookies on `file://` pages, so when you open the file straight off disk the local-storage copy is what remembers you; served over https, both work.
- Your sheet link is stored **in your own browser only**. Nothing is sent to GitHub, to me, or anywhere else — the hosted page is static files with no backend to send it to.

---

## Disclaimers

**Not investment advice.** This page is a personal viewer for your own spreadsheet. Nothing here is a recommendation to buy, sell or hold any security, and it takes no account of your circumstances or objectives. Do your own research and speak to a qualified adviser before making investment decisions.

**Figures are delayed and may be wrong.** Values come from your Google Sheet, which gets them from Google Finance via the `GOOGLEFINANCE` function — typically delayed 15–20 minutes and in some cases longer. They are loaded once when the page opens and do not update until you press Refresh, so what you see may be stale. Quotes can be incomplete, interrupted or simply incorrect, and are not suitable for trading decisions. Check with your broker or the exchange before acting on anything shown here.

**Open and closed markets are a guess.** Rows are greyed out when the exchange in the ticker is outside its regular trading hours, worked out from your computer's clock. Public holidays and unscheduled halts are not known here, so a market shut for a holiday will still look open, and auctions or extended-hours trading are not reflected.

**Amounts are not converted.** Each row is shown in whatever currency your sheet reports, so totals across currencies are not comparable. Daily change and percentage are taken from your sheet, or derived from the values it provides.

**No affiliation, no warranty.** Market data belongs to Google and the relevant exchanges and is subject to their terms; this page is not affiliated with, endorsed by or supported by Google, any exchange or any data provider. It is provided as is, without warranty of any kind, and is intended for personal use. Use of the data is at your own risk.

Redistributing exchange data has rules of its own that go beyond a disclaimer. If you plan to publish this page publicly rather than use it yourself, get advice first.

---

## Links

- **Live page** — https://berlotti.github.io/classic-Gfinance/
- **Source** — https://github.com/berlotti/classic-Gfinance
- **Issues and suggestions** — https://github.com/berlotti/classic-Gfinance/issues

If it's useful to you and you'd like to say thanks, you can [buy me a coffee](https://buymeacoffee.com/berlotti). Entirely optional — the page is free and always will be.
