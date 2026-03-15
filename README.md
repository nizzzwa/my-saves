# LinkedIn Saves Exporter + My Saves

A two-part toolkit for exporting and organising your LinkedIn saved posts.

- **Chrome Extension** — auto-scrolls your LinkedIn saved posts page and exports everything to JSON
- **My Saves Web App** — import that JSON and manage your posts with buckets, search, AI categorisation, and more

---

## Table of Contents

- [Chrome Extension](#chrome-extension)
  - [Features](#extension-features)
  - [Installation](#installation)
  - [Usage](#usage)
  - [JSON Output Format](#json-output-format)
  - [Known Limitations](#known-limitations)
- [My Saves Web App](#my-saves-web-app)
  - [Features](#app-features)
  - [Deploy](#deploy)
  - [AI Categorisation](#ai-categorisation)
- [Tech Stack](#tech-stack)
- [Contributing](#contributing)

---

## Chrome Extension

A Manifest V3 Chrome extension that scrapes `linkedin.com/my-items/saved-posts/` and exports your posts as a JSON file.

### Extension Features

| Feature | Description |
|---|---|
| **Export All** | Scrolls the entire saved posts page from the top and captures everything |
| **Export Latest** | Stops when it hits your anchor post (newest post from last export) — no duplicates |
| **Export JSON** | Downloads `linkedin-saved-posts.json` to your machine |
| **Timeline Card** | Shows the newest and oldest post collected with a count in between |
| **Anchor System** | Saves `posts[0]` after every export as the stop point for the next incremental run |
| **Background Persistence** | Scrape state survives popup close mid-run via a service worker relay |

### Installation

> The extension is currently pending review on the Chrome Web Store. Once approved, install it directly from there.

To install manually in the meantime:

1. Clone or download this repo
2. Open `chrome://extensions` in Chrome
3. Enable **Developer mode** (top right toggle)
4. Click **Load unpacked** and select the `/extension` folder

### Usage

1. Navigate to `linkedin.com/my-items/saved-posts/` and make sure the page is loaded
2. Click the extension icon in your toolbar
3. Choose **Export All** (first time) or **Export Latest** (subsequent runs)
4. Wait for the scrape to complete — the popup shows live progress
5. Click **Export JSON** to download your posts

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

> `scrapeOrder` reflects DOM position top-to-bottom. `date` is always empty — LinkedIn does not expose save date in the DOM.

### Known Limitations

- **Author extraction is best-effort.** LinkedIn's DOM changes frequently. The extension uses `span[aria-hidden="true"]` and semantic class selectors, with a junk filter to remove noise like "Status is offline".
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

A single-file, zero-backend HTML app for importing, organising, and revisiting your LinkedIn saved posts. Everything runs in the browser using `localStorage` — no server, no build step.

### App Features

#### Import & Parsing
- Upload a JSON file exported by the Chrome extension
- Handles multiline post text, extracts authors correctly, deduplicates by URL and activity ID
- New posts land in **Imported / Uncategorised** with `bucket: null`

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
- Onboarding screen for new users
- Import modal with 3-step instructions
- Fonts: **Instrument Serif** (logo) + **DM Sans** (UI)

### Deploy

The app is a single `index.html` file. The easiest way to deploy it:

1. Push `index.html` to a GitHub repository
2. Go to **Settings → Pages**
3. Set source to **Deploy from branch: main / root**
4. Your app will be live at `https://<your-username>.github.io/<repo-name>`

### AI Categorisation

The app includes an AI categorise feature that calls the Anthropic API directly from the browser.

- Batches 30 posts per request
- Updates bucket counts in real time as batches complete
- Supports Cancel and Undo

To use it, you'll need an [Anthropic API key](https://console.anthropic.com). Enter it in the app settings when prompted.

> **Note:** Your API key is stored only in your browser's `localStorage` and is never sent anywhere except directly to `api.anthropic.com`.

### App File Structure

```
/
├── index.html              # Production deploy file (empty seed data)
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
| AI | Anthropic API (`claude-sonnet-4-5`) |
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
