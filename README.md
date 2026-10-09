# 303 Apparel — storefront (HTML, CSS, JavaScript)

This is the front-end-only build: everything runs from static files and browser
storage (localStorage), so it works by just opening the site — no server, PHP
or MongoDB needed yet.

## Open it
Open `index.html` in a browser, or serve the folder so paths resolve cleanly:

```bash
python3 -m http.server 8080   # from inside this folder
# then open http://localhost:8080
```

## Pages
- `index.html` — Landing page: hero, about, featured products, promo banner with
  countdown, promotional video
- `features.html` — Feature page
- `products.html` — Product catalog: search, category filter, sort, and
  **pagination** (8 per page) — the whole gallery lives in this one file
- `contact.html` — Contact form + company info
- `social.html` — "Cop the Drop" social media campaign mockups
- `admin.html` — Staff area (passcode: `a303admin`, set in `assets/js/admin.js`)

## Product photos and galleries
Each product's photos live in `assets/img/products/<slug>/1.jpg`, `2.jpg`, etc.
A product with more than one photo shows a photo-count badge on its card, and
its detail view opens a gallery with arrows, swipe-style dots and left/right
arrow-key support. Products with only one photo just show that photo, no
gallery controls. The catalog itself — names, prices, categories, stock,
featured flag, description, and the photo list — lives in `assets/js/data.js`.

## Admin
The admin page can add a product (with 1–8 photos), edit one, delete one,
toggle which ones are "Featured" (shown on the home page), and edit the promo
code. These are saved in the browser's localStorage, so they're local to
whichever browser/device the admin uses — this is the front-end stage; see
below for what changes once the database is wired up.

## Sample data
Five made-up customers are seeded into the admin's Customers tab the first
time a browser loads the site (`window.SEED_CUSTOMERS` in `assets/js/data.js`),
so that dataset isn't empty before anyone has actually checked out. Real
orders are added on top of this and never overwrite it.

## 404 page
`404.html` is a branded "page not found" screen. Netlify serves it
automatically for any unmatched URL once the site is deployed, no extra
configuration needed.

## The product catalog, and a note on browser caching
The storefront saves its catalog into each visitor's browser (localStorage) the
first time they load it, so admin changes stick around on return visits without
a database. That means if you edit `assets/js/data.js` later, a browser that
already has a saved catalog won't see the edit on its own — so `data.js` also
sets `window.SEED_VERSION`. **Bump that number any time you change
`SEED_PRODUCTS`** and every visitor's browser will refresh to the new list
automatically on their next load (this does discard anything they'd added
through the admin page, by design — the seed is meant to be the ground truth
whenever it changes). The admin page's own "Reset to the original catalog"
button does the same thing on demand.

## Promo
Code **15FF** gives 15% off and runs through **October 15, 2026**. Edit it
from the admin page, or change the default in `assets/js/store.js`
(`DEFAULT_PROMO`).

## Swapping in MongoDB later
All data access goes through `assets/js/store.js` (`window.Store`), with one
function per thing the site needs: `listProducts`, `getProduct`, `addProduct`,
`updateProduct`, `deleteProduct`, `getPromo`, `setPromo`, `placeOrder`,
`listCustomers`, `addMessage`, `listMessages`. Right now each one reads or
writes `localStorage`. When the MongoDB-backed API is ready, replace the body
of each function with a `fetch()` call to that API — the rest of the site
(`common.js`, `home.js`, `products.js`, `admin.js`, etc.) calls `Store.*` and
never touches storage directly, so nothing else needs to change.

## Brand assets
`assets/img/logo-mark-cream.png` (header, on navy) and `assets/img/logo-badge-navy.png`
(hero badge, on cream) come straight from the official logo files. The favicons
(`favicon-32.png`, `favicon-192.png`, `apple-touch-icon.png`, `favicon.ico`) are
cropped from the same navy mark. Brand colors (`--navy` #2a3f62, `--cream` #fef9ea)
in `assets/css/style.css` are sampled from those files, so any new art in the same
navy/cream should match automatically.

## Promotional video
Put your video file at `assets/video/promovid303.mp4` (rename it, or update
`VIDEO_FILE` in `assets/js/config.js`, if it uses a different extension).

`initVideoSystem()` in `assets/js/common.js` creates **two separate video
elements** — one for the floating widget, one for the dedicated section —
and neither is ever moved to a different parent. (An earlier version shared
one element and relocated it with `appendChild`, which looked fine on
desktop Chrome but broke on many phones and some other browsers: a playing
`<video>` that gets reparented mid-playback can lose its picture while the
audio keeps going, a known mobile-WebKit/Chromium quirk. Two independent
elements, handed off by pausing one and syncing + resuming the other, avoids
it entirely.) The two spots:

- **Floating mini player.** Visible from the moment the page loads (top of
  the viewport, no scrolling needed), draggable by its title bar, resizable
  by the handle in its corner or the arrow keys — always 4:3, between 160px
  and 420px wide. Edit `MIN_W`/`MAX_W` in `initVideoSystem()` to change the
  limits. The ✕ pauses it and sends it to the dedicated section below for
  the rest of this page view (it comes back on a reload).
- **Dedicated section** further down the landing page — where the same
  video lands the moment you scroll it into view (so there's never two
  copies playing at once, and the floating one never plays silently out of
  sight). Also 4:3, independently resizable (260px–820px) with its own
  handle. Edit `MIN_W`/`MAX_W` in `assets/js/home.js` to change its limits.

## Things to fill in
- **Promotional video:** set `VIDEO_EMBED_URL` or `VIDEO_FILE` in
  `assets/js/config.js`.
- **Company details:** phone, email and hours in the same file are
  placeholders.
- **Social links:** replace the `#` placeholders in `config.js`.
- **Admin passcode:** change `ADMIN_KEY` in `assets/js/admin.js` before
  sharing this anywhere public — right now it's a plain string in the code,
  fine for a class demo, not for a real deployment.
