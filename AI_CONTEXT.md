# AI_CONTEXT.md — Family Video Gallery

> **Read this file first before making any changes to this project.**
> Do not analyze the entire project unless the requested change specifically requires it. Inspect only the relevant files after understanding this document.

---

## 1. PROJECT OVERVIEW

- **Project name:** Family Video Gallery
- **Purpose:** A private family video gallery that displays YouTube videos in a premium, cinematic interface.
- **Tech stack:** React + Vite + JavaScript + Vanilla CSS
- **Main functionality:** Fetches a list of YouTube URLs from `public/videos.json`, automatically generates thumbnails, fetches real video titles, and plays videos in a modal using YouTube's official embed player.
- **Design concept:** Dark, elegant, cinematic, professional. Feels like a premium private photo/video archive — NOT a generic dashboard or admin panel.

---

## 2. PROJECT STRUCTURE

```
family-video-gallery/
├── public/
│   └── videos.json              # Data source — array of video entries
├── src/
│   ├── components/
│   │   ├── Header.jsx           # Hero header ("Family Memories" title, subtitle, lock icon)
│   │   ├── Header.css
│   │   ├── VideoGallery.jsx     # Renders the responsive grid of VideoCards
│   │   ├── VideoGallery.css
│   │   ├── VideoCard.jsx        # Individual video thumbnail card (fetches title, shows fallback)
│   │   ├── VideoCard.css
│   │   ├── VideoModal.jsx       # Fullscreen modal with YouTube iframe player
│   │   ├── VideoModal.css
│   │   ├── PrivacyOverlay.jsx   # Blurs/hides content when user switches tabs
│   │   ├── PrivacyOverlay.css
│   │   ├── Notification.jsx     # Toast notification for blocked actions
│   │   └── Notification.css
│   ├── utils/
│   │   ├── youtube.js           # YouTube URL parsing, thumbnail URL, embed URL generation
│   │   └── security.js          # Casual copy deterrent event listeners
│   ├── App.jsx                  # Root component — state management, data fetching, layout
│   ├── App.css
│   ├── index.css                # Global styles, CSS variables, design tokens
│   └── main.jsx                 # Vite entry point
├── index.html                   # HTML shell (includes Google Fonts)
├── package.json
├── vite.config.js
└── README.md
```

---

## 3. DATA SOURCE

**File:** `public/videos.json`

**Format:**
```json
[
  {
    "url": "https://www.youtube.com/watch?v=VIDEO_ID",
    "title": "Optional fallback title"
  },
  {
    "url": "https://youtube.com/shorts/SHORT_ID"
  }
]
```

**Rules:**
- `url` — **Required.** Must be a valid YouTube URL.
- `title` — **Optional.** Used as a fallback title when the real YouTube title cannot be fetched.
- Plain string URLs are also supported for backward compatibility (e.g. `"https://youtu.be/ID"`).
- `VideoGallery.jsx` normalizes both formats (string or object) before rendering.

**Title priority chain:**
1. Real YouTube title (fetched from noembed.com)
2. `title` field from `videos.json` (fallback for private/unavailable videos)
3. `"Family Memory XX"` (final fallback, auto-numbered)

**Adding a video:** Add a new entry to the JSON array.
**Removing a video:** Delete its entry from the JSON array.
**No React code changes are needed to manage videos.**

---

## 4. YOUTUBE URL SUPPORT

The parser in `src/utils/youtube.js` (`getYouTubeData`) supports these URL formats:

| Format | Example |
|---|---|
| Standard | `https://www.youtube.com/watch?v=VIDEO_ID` |
| Without www | `https://youtube.com/watch?v=VIDEO_ID` |
| Short URL | `https://youtu.be/VIDEO_ID` |
| Embed URL | `https://www.youtube.com/embed/VIDEO_ID` |
| Shorts | `https://youtube.com/shorts/VIDEO_ID` |
| Shorts with params | `https://youtube.com/shorts/VIDEO_ID?feature=share` |

**How extraction works:**
- Uses the `URL` constructor to parse the URL.
- Checks the hostname (`youtu.be` vs `youtube.com`).
- Checks the pathname to determine the type (`/watch`, `/embed/`, `/shorts/`).
- For `/watch`, reads the `v` query parameter.
- For `/shorts/` and `/embed/`, extracts the ID from the pathname.
- Query parameters like `?feature=share` are safely ignored.

**Return value:** `{ id: string, isShort: boolean }` or `null` if the URL is invalid.

---

## 5. VIDEO TYPES

- **Normal YouTube videos:** Detected when the URL is `/watch`, `/embed/`, or `youtu.be`. `isShort` is `false`. Presented in **16:9 aspect ratio**.
- **YouTube Shorts:** Detected when the URL contains `/shorts/`. `isShort` is `true`. Presented in **9:16 vertical/portrait aspect ratio**.

The modal (`VideoModal.jsx`) uses the `isShort` flag to apply different CSS classes:
- Normal: `video-container` (padding-top: 56.25% for 16:9)
- Shorts: `video-container shorts-container` (padding-top: 177.77% for 9:16) and `modal-content shorts-modal` (max-width: 450px)

---

## 6. THUMBNAILS

**Generation:** Thumbnails are auto-generated using YouTube's public thumbnail URL:
```
https://img.youtube.com/vi/VIDEO_ID/hqdefault.jpg
```
This is handled by `getYouTubeThumbnail()` in `src/utils/youtube.js`.

**Thumbnail failure does NOT mean the video is unavailable.** Private YouTube videos may not expose thumbnails publicly. When a thumbnail image fails to load (`onError`), the card shows a premium fallback containing:
- Lock icon
- Play icon
- "Private Family Video"
- "Open Video"

The card remains **fully clickable**. Only YouTube's official player determines whether the signed-in Google account can actually watch the video.

---

## 7. PRIVATE VIDEO MODEL

YouTube Private permissions are the **actual** video authorization mechanism.

**The frontend does NOT and must NOT:**
- Bypass YouTube permissions
- Download or extract videos
- Expose API keys, OAuth tokens, or secrets
- Store passwords or access tokens
- Use third-party download/extraction services
- Attempt to determine video availability by testing thumbnail URLs

Video IDs are identifiers, not secrets. The YouTube embed player naturally handles access restriction — if a user doesn't have permission, YouTube shows its own error.

---

## 8. TITLE LOGIC

Implemented in `VideoCard.jsx`:

```
YouTube title (fetched via noembed.com oEmbed proxy)
  ↓ (if unavailable)
JSON fallback title (from videos.json "title" field)
  ↓ (if unavailable)
"Family Memory XX" (auto-numbered from array index)
```

**How the YouTube title is fetched:**
- `VideoCard.jsx` calls `https://noembed.com/embed?url=https://www.youtube.com/watch?v=VIDEO_ID`
- noembed.com is a **CORS-friendly** oEmbed proxy (YouTube's own oEmbed endpoint blocks browser CORS requests)
- No API key is needed
- If the fetch succeeds and returns a title, that title is used
- If it fails (private video, network error), the fallback chain applies

---

## 9. SECURITY / PRIVACY FEATURES

All of these are **casual deterrents only, NOT real security.** Implemented in `src/utils/security.js`:

| Feature | Implementation |
|---|---|
| Right-click blocking | `contextmenu` event `preventDefault()` |
| Keyboard shortcut blocking | Ctrl+S, Ctrl+U, Ctrl+C, Ctrl+Shift+I, F12 intercepted via `keydown` |
| Image/link drag prevention | `dragstart` event on IMG/A tags |
| CSS text selection prevention | `.unselectable` class with `user-select: none` |
| Thumbnail watermark | "PRIVATE FAMILY ARCHIVE" overlay on each card |
| Privacy overlay on tab switch | `PrivacyOverlay.jsx` listens to `visibilitychange`; blurs content when `document.visibilityState === 'hidden'` |
| Notification toast | Shows a brief message when a blocked action is attempted |

**These are deterrents, not real security.** A determined user can always use DevTools, screen recording, or inspect network traffic. Frontend JavaScript cannot prevent this.

**Never implemented / never implement:**
- Infinite loops, debugger traps, forced reloads
- Browser history manipulation
- Window trapping or browser crashes
- Malicious anti-debugging scripts

---

## 10. VIDEO PLAYER

**Component:** `VideoModal.jsx`

**How it works:**
1. User clicks a `VideoCard` → `App.jsx` sets `activeVideo` state with the `{ id, isShort }` object.
2. `VideoModal` renders as a fixed overlay with backdrop blur.
3. The YouTube iframe is **only created when the modal opens** (lazy loading). It is destroyed when the modal closes.
4. Embed URL format: `https://www.youtube.com/embed/VIDEO_ID?autoplay=1&rel=0`

**Closing the modal:**
- Click the X button
- Press ESC key
- Click the backdrop (outside the player)

**Behavior:**
- Background scrolling is locked (`document.body.style.overflow = 'hidden'`)
- Modal auto-focuses for keyboard accessibility
- Footer shows: "Protected by YouTube Permissions. Only authorized accounts can view this video."

**Aspect ratios:**
- Normal video: 16:9 (CSS `padding-top: 56.25%`)
- YouTube Short: 9:16 (CSS `padding-top: 177.77%`, modal max-width: 450px)

---

## 11. IMPORTANT UI RULES

The current design language is:
- **Premium and cinematic** — dark background (#0a0a0a), gold accent (#d4af37)
- **Professional** — not a toy, not a dashboard
- **Family-memory gallery** — warm, elegant, personal
- **Responsive** — CSS grid auto-fill with minmax, adapts to desktop/tablet/mobile
- **Elegant typography** — Playfair Display for headings, Inter for body (loaded from Google Fonts)
- **Smooth animations** — CSS transitions on hover, modal open/close, card lift effects

**Future updates must preserve the existing design unless explicitly requested otherwise.**

---

## 12. IMPORTANT DEVELOPMENT RULES

Future AI assistants should:
- **Modify only the necessary files** for the requested change
- **Avoid rebuilding the project** from scratch
- **Preserve all existing features** — gallery, modal, privacy, security, error states, loading states
- **Preserve existing UI** — dark theme, animations, typography, layout
- **Avoid changing the data format** (`videos.json` structure) without asking the user first
- **Avoid introducing unnecessary npm dependencies**
- **Never expose secrets** — no API keys, no tokens, no passwords in frontend code
- **Never remove existing privacy/security features** accidentally
- **Never implement malicious anti-debugging** (infinite loops, debugger traps, etc.)

---

## 13. HOW TO ADD A VIDEO

Edit `public/videos.json` and add a new entry:

```json
{
  "url": "https://www.youtube.com/shorts/VIDEO_ID",
  "title": "Wedding Moments"
}
```

- `url` is **required**
- `title` is **optional** (used as fallback if YouTube title cannot be fetched)
- Plain string URLs also work: `"https://youtu.be/VIDEO_ID"`

No React code changes needed.

---

## 14. FUTURE UPDATE INSTRUCTIONS

> **Read this AI_CONTEXT.md first before making changes.**
> Do not analyze the entire project unless the requested change requires it.
> Inspect only the relevant files after understanding this document.
> Check the current implementation status below before assuming what exists or doesn't exist.

---

## 15. CURRENT IMPLEMENTATION

### What is currently implemented:

- [x] React + Vite project structure
- [x] `public/videos.json` data source (object format with optional title, plain strings also supported)
- [x] YouTube URL parser supporting: standard, short, embed, and Shorts URLs
- [x] Automatic Shorts detection from URL (`isShort` flag)
- [x] YouTube thumbnail auto-generation via `img.youtube.com`
- [x] Real YouTube title fetching via noembed.com (CORS-friendly)
- [x] Title fallback chain: YouTube → JSON → "Family Memory XX"
- [x] Responsive video gallery grid (CSS grid, auto-fill)
- [x] Cinematic dark theme with gold accents
- [x] Google Fonts: Playfair Display (headings), Inter (body)
- [x] Video cards with hover animations, play button overlay, watermark
- [x] Premium fallback card for failed thumbnails (lock icon, play icon, "Private Family Video")
- [x] Video modal with YouTube iframe embed (lazy-loaded on click only)
- [x] Modal: ESC close, backdrop click close, X button close, scroll lock
- [x] Normal video: 16:9 player aspect ratio
- [x] Shorts: 9:16 vertical player (portrait modal on desktop, full-width on mobile)
- [x] Loading state with spinner
- [x] Error state for failed `videos.json` fetch
- [x] Empty state when no videos exist
- [x] Invalid URL placeholder card
- [x] Privacy overlay when tab is hidden (`visibilitychange`)
- [x] Casual copy deterrents (right-click, keyboard shortcuts, drag prevention)
- [x] Toast notification for blocked actions
- [x] CSS `.unselectable` class
- [x] README.md with install, usage, and security disclaimer

### What is NOT implemented:

- [ ] Light/dark theme toggle (dark theme only)
- [ ] Search or filter functionality
- [ ] Video categories or tags
- [ ] User authentication
- [ ] Backend/server
- [ ] Database
- [ ] YouTube Data API integration
