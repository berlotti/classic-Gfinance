# Classic Watchlist

A single-file HTML page that shows your stock watchlists in the style of the **old Google Finance** — ticker badge, company name, price, daily change, and a coloured percentage pill, one row per holding.

It reads its data from a Google Sheet that you own. No account, no API key, no server, no build step. Run it on [Github itself](https://berlotti.github.io/classic-Gfinance) or [download the file](https://github.com/berlotti/classic-Gfinance/tree/main/docs), open it in a browser, done.

---

## Why this exists

Google retired the old Google Finance layout in favour of a AI-heavy design built around news, discovery and charts. That is a reasonable product for browsing. It is a worse product for the thing a lot of people actually used the old one for: **glancing at a list of tickers and seeing, in one screen, what moved today.**

The old watchlist put roughly a dozen holdings above the fold in a dense table you could read in two seconds. The replacement spends that space on other things.

This page brings that view back, with three deliberate differences:

- **Your data lives in your spreadsheet**, not in a product that can be redesigned out from under you. If this page ever stops working, your watchlist is still a Google Sheet.
- **It loads once.** No live ticking, no polling, no attention-grabbing. Open it, read it, close it.
- **It is one HTML file.** You can read the whole thing, use it on [Github pages])https://berlotti.github.io/classic-Gfinance), or host it yourself, or keep it on your desktop forever.

There is no official Google Finance API — it was deprecated in 2011 and shut down in 2012. The only supported way to get Google's finance data is the `GOOGLEFINANCE()` function inside Google Sheets, which is exactly what this setup uses.

---

## Quick start (2 minutes)

1. Save `index.html` somewhere convenient and open it in your browser. Rename it if you want.
2. Press **Demo sheet** to see it working with an example spreadsheet.
3. When you want your own data, follow *Making your own sheet* below, then paste your sheet's link into the box at the top and press **Load**.

The page remembers the last sheet you used, so from then on it just opens to your watchlist. Nothing is shown on a fresh browser until you load a sheet.

---

## Making your own sheet

The easiest route is to copy the demo sheet and edit it, so the layout is already correct.

### 1. Copy the demo sheet

Open the demo sheet:

```
https://docs.google.com/spreadsheets/d/1UvTu-ZDs8euxx4h5qWev9wBgc-MBPdy7ZzE9ePHGEh0/edit?usp=sharing
```

Then **File → Make a copy**. You now own the copy and can edit it freely.

### 2. Put your own tickers in

In each tab, replace the tickers in **column A**. Every other column is a formula that follows column A, so once a ticker is in place the rest fills itself in. To add rows, drag the formulas down from the row above.

Ticker format is `EXCHANGE:TICKER` — the same thing Google Finance shows in its URL:

| What | Example |
|---|---|
| US stock | `NASDAQ:AAPL`, `NYSE:V` |
| European stock | `AMS:ASML`, `ETR:SAP`, `LON:SHEL` |
| Hong Kong | `HKG:0700` |
| Index | `INDEXSP:.INX`, `INDEXDJX:.DJI`, `INDEXDB:DAX` |
| Crypto / FX | `CURRENCY:BTCUSD`, `CURRENCY:EURUSD` |

### 3. Share it so the page can read it

**Share → General access → Anyone with the link → Viewer → Copy link.**

Viewer is enough. The page only ever reads; it never writes to your sheet.

> Anyone who has the link can see this sheet. Don't put anything private in it — share counts, cost prices and account numbers do not belong in a link-shared document.

You do **not** need *File → Publish to web*. The ordinary share link is what this page expects.

### 4. Load it

Paste the link into the box at the top of the page and press **Load** (or just press Enter). A bare sheet ID works too.

Click the title above the box to give the sheet a name — Google's data endpoint doesn't expose the spreadsheet's own title, so this is a label you set. It's remembered per sheet.

---

## Sheet requirements

These are the rules the page relies on. The demo sheet already follows all of them.

### Tabs must be named `1`, `2`, `3`, …

Each tab is one watchlist. Name the tabs with plain numbers and nothing else.

The page asks Google for tab `1`, tab `2` and so on until it stops finding new ones. Up to **24** tabs are supported. A gap is fine — if you delete tab `2` and keep `3`, tab `3` still appears.

> **Why numbers?** Asking for a tab that doesn't exist doesn't return an error — Google quietly serves tab 1 again. Numbered tabs let the page probe predictably and spot that fallback, which is how it knows where your lists end.

### Cell A1 holds the list name

The first row of each tab is the title row: put the name of that watchlist in **A1** and leave the rest of row 1 empty. That name becomes the tab label in the page.

Row 1 must contain **no numbers**, or it will be treated as data instead of a title.

### Data starts in row 2, with no header row

Do **not** add a `Symbol | Price | Change` header row. The page works out what each column is from the data itself, and a row of text where it expects numbers breaks that. The A1 title row is the only non-data row allowed.

### Columns

The page identifies columns by their content rather than by position, but there are two rules it can't work around:

- **The ticker must be the leftmost text column.** The first column that isn't numbers is taken to be the ticker.
- **Numeric columns are read left to right as price, then change, then % change.** Two numbers alone means price and change, never price and percentage.

Within those rules you can arrange things how you like — the demo layout below is simply the sensible one.

| Column | Required | Notes |
|---|---|---|
| Ticker | **Yes** | Text, e.g. `NASDAQ:AAPL`. Must be the first text column |
| Price | **Yes** | A number. The first numeric column |
| Daily change | No | The second numeric column |
| Daily change % | No | The third numeric column. Worked out from the change if absent |
| Company name | No | Text, after the ticker. Falls back to the ticker itself |
| Currency | No | A three-letter code like `USD`. Without it, prices show as plain numbers |

If you supply the change but not the percentage, the percentage is calculated for you, and vice versa — but only when the column that *is* present sits in its expected position.

The formulas the demo sheet uses, in `B2:F2`, dragged down:

```
=GOOGLEFINANCE($A2,"price")
=GOOGLEFINANCE($A2,"change")
=GOOGLEFINANCE($A2,"changepct")
=GOOGLEFINANCE($A2,"name")
=GOOGLEFINANCE($A2,"currency")
```

If a ticker occasionally returns `#N/A`, wrap it: `=IFERROR(GOOGLEFINANCE($A2,"price"),"")`.

Blank rows and `#N/A` rows are ignored, so it's fine to leave formulas sitting in empty rows below your holdings.

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

**Greyed-out rows** are on an exchange that isn't trading right now, worked out from the exchange in the ticker and your computer's clock. It knows lunch breaks in Hong Kong, Tokyo, Shanghai and Singapore. It does **not** know public holidays — see the disclaimers.

Your sheet, chosen tab, sort order, theme and sheet name are remembered locally in your browser. Nothing is sent anywhere.

---

## Troubleshooting

**"Couldn't read the sheet" / "Google returned a web page instead of data"**
The sheet isn't shared publicly. Go to **Share → General access → Anyone with the link → Viewer**. This is by far the most common cause.

**"Couldn't find a price column in the sheet"**
Usually a header row — remove it, as only A1's title row is allowed. Otherwise the price column contains text rather than numbers; check the cells aren't formatted as plain text.

**Prices, changes and percentages in the wrong columns**
The ticker has to be the leftmost text column, and numbers are read left to right as price, change, percentage. A currency or name column to the left of the ticker will be mistaken for the ticker itself.

**Only one list shows, or a tab is missing**
Tab names must be plain numbers. `Sheet1`, `Tech`, `1.` and `01` won't be found — rename to `1`, `2`, `3`.

**Two lists show as one**
If a tab contains exactly the same tickers *and* the same A1 name as tab 1, the page can't distinguish it from Google's silent fallback and hides it. Change the name or the contents.

**Numbers look stale**
`GOOGLEFINANCE` is delayed 15–20 minutes, and the page loads once. Press **Refresh**. Google also caches these responses for a few minutes.

**Everything is greyed out**
Check your computer's clock and time zone. If your browser has no time-zone data, nothing is greyed rather than guessing.

**Nothing at all on first open**
That's intended. Paste a link or press **Demo sheet**.

---

## How it works

The page asks Google's spreadsheet visualisation endpoint for each tab as JSON:

```
https://docs.google.com/spreadsheets/d/<SHEET_ID>/gviz/tq?tqx=out:json&headers=0&sheet=1
```

It loads that with a `<script>` tag rather than `fetch()`. This matters: Google doesn't send CORS headers on that endpoint, so a normal `fetch()` from a page opened off your disk is blocked by the browser and fails with *"Failed to fetch"*. Script tags aren't subject to CORS, and the endpoint will wrap its reply in a function call for exactly this purpose.

Consequences worth knowing:

- The only network request the page makes is to `docs.google.com`, for your sheet. No proxies, no CDNs, no analytics, no third parties.
- It works from a `file://` path, a USB stick, or any static web host.
- Preferences are stored in a cookie *and* in local storage. Browsers refuse cookies on `file://` pages, so when you open the file directly the local-storage copy is what actually remembers you.

---

## Disclaimers

**Not investment advice.** This page is a personal viewer for your own spreadsheet. Nothing here is a recommendation to buy, sell or hold any security, and it takes no account of your circumstances or objectives. Do your own research and speak to a qualified adviser before making investment decisions.

**Figures are delayed and may be wrong.** Values come from your Google Sheet, which gets them from Google Finance via the `GOOGLEFINANCE` function — typically delayed 15–20 minutes and in some cases longer. They are loaded once when the page opens and do not update until you press Refresh, so what you see may be stale. Quotes can be incomplete, interrupted or simply incorrect, and are not suitable for trading decisions. Check with your broker or the exchange before acting on anything shown here.

**Open and closed markets are a guess.** Rows are greyed out when the exchange in the ticker is outside its regular trading hours, worked out from your computer's clock. Public holidays and unscheduled halts are not known here, so a market shut for a holiday will still look open, and auctions or extended-hours trading are not reflected.

**Amounts are not converted.** Each row is shown in whatever currency your sheet reports, so totals across currencies are not comparable. Daily change and percentage are taken from your sheet, or derived from the values it provides.

**No affiliation, no warranty.** Market data belongs to Google and the relevant exchanges and is subject to their terms; this page is not affiliated with, endorsed by or supported by Google, any exchange or any data provider. It is provided as is, without warranty of any kind, and is intended for personal use. Use of the data is at your own risk.

Redistributing exchange data has rules of its own that go beyond a disclaimer. If you plan to publish this page publicly rather than use it yourself, get advice first.
