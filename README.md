# LinkedIn Saves Exporter + My Saves

A two-part free toolkit for exporting and organising your LinkedIn saved posts.

- **Chrome Extension** — auto-scrolls your LinkedIn saved posts page and exports everything to JSON
- **My Saves Web App** — import that JSON and manage your posts with buckets, search, filters, and more

No backend. No account. No cost. Everything runs in your browser.

---

## How It Works

```
  LinkedIn Saved Posts
        ↓
  Chrome Extension  ←  install once, run periodically
        ↓
  JSON file download
        ↓
  My Saves web app  ←  import, organise, revisit
```

1. Install the Chrome extension
2. Go to your LinkedIn saved posts and hit **Export All** → download the JSON
3. Upload the JSON to My Saves
4. Organise your posts into buckets, search, star, pin, archive
5. Run the extension periodically → click **Export Latest** to pull in only new saves without duplicates

---

## Chrome Extension

[![Available in the Chrome Web Store](https://img.shields.io/badge/Chrome%20Web%20Store-Install-blue?logo=google-chrome)](https://chromewebstore.google.com/detail/linkedin-saves-exporter/gjodnbfdffddoliaimeoaanjfldemnmn)

👉 **[Install from Chrome Web Store ↗](https://chromewebstore.google.com/detail/linkedin-saves-exporter/gjodnbfdffddoliaimeoaanjfldemnmn)**

### Features

| Feature | Description |
|---|---|
| **Export All** | Scrolls the entire saved posts page from the top and captures everything |
| **Export Latest** | Stops when it hits your anchor post (newest post from last export) — no duplicates |
| **Export JSON** | Downloads `linkedin-saved-posts.json` to your machine |
| **Timeline Card** | Shows the newest and oldest post collected with a count in between |
| **Anchor System** | Saves `posts[0]` after every export as the stop point for the next incremental run |
| **Background Persistence** | Export state survives popup close mid-run via a service worker relay |

### JSON Output Format

Each post in the exported file follows this shape:

```json
{
  "id": "https://linkedin.com/feed/update/urn:li:activity:...",
  "url": "https://linkedin.com/feed/update/urn:li:activity:...",
  "title": "Post text truncated to 300 characters...",
  "author": "Author Name",
  "date": "",
  "scrapeOrder": 0
}
```

> `scrapeOrder` is the field name in the JSON output reflecting DOM position top-to-bottom. `date` is always empty — LinkedIn does not expose save date in the DOM.

### Known Limitations

- **Author extraction is best-effort.** LinkedIn's DOM changes frequently. The extension uses `span[aria-hidden="true"]` and semantic class selectors with a junk filter to remove noise.
- **No save date.** LinkedIn doesn't include the date a post was saved in the page DOM.
- **LinkedIn DOM changes.** If LinkedIn ships a major redesign, selectors may need updating.

### Extension File Structure

```
extension/
├── manifest.json       # MV3, permissions: activeTab, scripting, storage, tabs
├── background.js       # Service worker — relays PROGRESS/DONE messages to storage
├── content.js          # Injected script — scrolls, extracts, sorts posts
├── popup.html          # Extension UI (4 states: first-time, running, done, returning)
└── popup.js            # Popup logic and state management
```

---

## My Saves Web App

👉 **[Open My Saves ↗](https://nizzzwa.github.io/my-saves)**

A single-file, zero-backend HTML app for importing, organising, and revisiting your LinkedIn saved posts. Everything runs in the browser using `localStorage` — no server, no build step, no account required.

### Features

#### Import & Parsing
- Upload a JSON file exported by the Chrome extension
- Handles multiline post text, extracts authors correctly, deduplicates by URL and activity ID
- New posts land in **Uncategorised** — existing posts are untouched
- Run periodically with **Export Latest** to pull in only new saves

#### Buckets (Sidebar)
- **System buckets:** Imported, Uncategorised, Archived, Starred, Pinned
- **Custom buckets** with emoji + name, draggable to reorder
- Per-bucket menu (⋯): rename, delete with undo toast, move or delete contained saves
- Live post counts per bucket

#### Cards (Main Area)
- Responsive grid layout (single column on mobile)
- Per-card actions: Mark Read, Archive, Star, Pin, Add Notes, Copy link, Open URL
- Bucket selector dropdown per card
- Filter by Unread / All / Read
- Sort by Newest / Oldest
- Search by name, author, or keywords

#### Animations
- **Mark Read** — Thanos particle disintegration effect; particles fly toward the Read button
- **Archive** — card clips upward revealing a folder icon; surrounding space collapses

#### UI / UX
- Dark mode default, light mode toggle (persisted in `localStorage`)
- Fully mobile responsive — sidebar slides in/out
- Onboarding screen for new users with step-by-step instructions
- Fonts: **Instrument Serif** (logo) + **DM Sans** (UI)

### App File Structure

```
/
├── index.html              # Production deploy file
└── saves-organizer.html    # Dev file with seed data for testing
```

#### Key `localStorage` Keys

| Key | Contents |
|---|---|
| `mysaves-v7` | All posts, buckets, and app state |
| `mysaves-theme` | `'light'` or `'dark'` |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Chrome Extension | Manifest V3, vanilla JS, service worker |
| Web App | Single-file HTML, vanilla JS, CSS |
| Storage | `localStorage` (no backend) |
| Analytics | PostHog |
| Fonts | Google Fonts — Instrument Serif, DM Sans |
| Deployment | GitHub Pages |

---

## Design

| Element | Value |
|---|---|
| Extension background | `#1A1814` (dark charcoal) |
| Extension CTA | `#C8410B` (orange) |
| Extension text | Cream |
| Web app accent | `#F0A520` (amber) on near-black |

---

## Contributing

Issues and PRs are welcome. If LinkedIn's DOM changes break author extraction, the relevant selectors are in `content.js` — look for the `extractAuthor` function.
