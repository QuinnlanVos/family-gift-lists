# The Giving Circle

Small, mobile-first family gift coordination site. Static HTML/CSS/JavaScript; no build step, no accounts, no trackers or third-party fonts. No repository or deployment is required to preview it.

## Run locally

```sh
cd /workspace/family-gift-lists
python3 -m http.server 8784 --bind 127.0.0.1
```
Open `http://127.0.0.1:8784`. Hosting must use HTTPS for PIN hashing (`crypto.subtle`); localhost also works. Publish only `index.html`, `style.css`, `app.js`, `import.js`, `config.js`, and optionally this README. Do not publish test dependencies, screenshots, test reports, or backup data.

## How it works

- Choose or add a name; the device remembers it. An optional salted PBKDF2-SHA256 4-digit PIN (100,000 iterations) deters casual name-switching. No raw PIN is stored.
- My wishlist holds a wishlist link and up to 100 individual wishes. Owners can edit/delete their items, paste up to 20 HTTPS links at once, and add title/note/price details.
- Other people can reserve each item as Main or Extra, view who reserved it, and undo only their own reservations. Free-text off-list gift notes are visible to everyone except their recipient.
- The recipient's own reservations and off-list notes never appear in their rendered interface, including their My wishlist page. This is UI spoiler prevention, NOT access control.
- Amazon wishlists open in a separate tab. Amazon contents are **not** automatically fetched or synchronized. The paste importer accepts copied wishlist text and clipboard HTML, extracts possible names/prices/ASIN product links, shows a checkbox preview and skips duplicates. It never signs in to or fetches Amazon. Select-all copies vary by device and may omit prices/links or include unrelated text: review the preview, uncheck mistakes, and edit saved wishes. Missing links are left blank. Up to 100 preview items, 100,000 text characters, 250,000 HTML characters. Duplicate checking uses normalized titles and canonical Amazon ASIN links. All copied markup is parsed inertly, never rendered. Add individual gifts here for reliable reservation coordination. The site cannot detect a duplicate purchase made solely on Amazon.
- Inputs are bounded and HTML-escaped. All links must be HTTPS without embedded credentials; outbound links use `noopener noreferrer` and no referrer. Names max 40 chars, titles 120, notes 400, off-list notes 300, prices 30, URLs 1500. Up to 60 people and 10,000 operations.

## Shared store

`config.js` contains the production key:

`gift-family-61ccc2a87ea925afd0bfe54664ecfdd0`

The TextDB endpoint is `https://textdb.dev/api/data/<key>`.

This is not a local-only data app: names, wishlists, gifts, claims, notes and PIN hashes are stored as public-readable plaintext JSON with TextDB, separate from the static host. No payments or login credentials should ever be entered. Anyone who can open the app can find its key and retrieve or change all stored data. A 4-digit PIN is guessable and bypassable; it is not authentication, encryption, or a secure secret.

Changes are append-only UUID operations merged into a JSON document. Every write reads the latest document, unions known operations, writes, and rereads to confirm (up to five retries with randomized backoff). Active clients also reconcile every five seconds. Operations are retained locally to repair lost updates. Competing reservations resolve deterministically to the earliest operation; all clients converge after polling. TextDB has no atomic compare-and-swap, so strict transactional guarantees are impossible: tell family to check their reservation again before buying, especially after simultaneous clicks or connection problems. Active devices can replay old operations after someone wipes the server; use a fresh random key for a new gift day or a complete reset.

TextDB is a free no-account hobby service with no promised availability, durability, privacy, or retention. There is no production data in the app files. A private backup repository would reduce convenient accidental spoilers, but would not make this public shared store private. A future truly private version requires authenticated server-side recipient authorization rather than a static shared JSON document.

## Tests

```sh
npm install --prefix test
node test/e2e.cjs
```

The test serves the site on loopback, substitutes a unique **test-only** key without changing `config.js`, and uses three fresh isolated headless Chromium contexts (Alice, Bob, Charlie). It verifies real TextDB read/write and CORS, main and extra claims, recipient spoiler hiding, third-person visibility, off-list notes, unclaim rules, bulk add/edit/delete, HTML escaping, remembered identities, correct/incorrect PIN handling, plaintext-PIN absence, mobile overflow, realistic Amazon copied-text parsing, checkbox selection, duplicate skipping, and HTML-clipboard import. Test data is cleared in a `finally` block after all browser contexts close. Screenshots show fictional test people only. `test-report.json` records results. Production data is never used in tests.
