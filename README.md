# Refloow-Eleblocker

Have you ever opened a website and run into a persistent banner, promotional box, or out-of-place element blocking your view and disrupting the layout? Some of these elements have no dismiss button, and others return every time you reload the page.

Refloow-Eleblocker is a free, open-source browser extension that lets you select an unwanted element once and keep it hidden on every future visit. Pages load without the blocked elements from the start, avoiding both visual flicker and layout shift.

<img width="466" height="502" alt="image" src="https://github.com/user-attachments/assets/4ffbc8a6-5b34-4a02-96ad-dbc607f047f0" />


## Features

- **Point-and-Click Element Picker:** Highlight any element directly on a page and click to generate a CSS selector and save the rule immediately.
- **Pre-Paint Blocking:** Targeted elements are hidden before the browser renders the page, rather than loading first and disappearing afterward.
- **Zero Layout Shift Options:** Choose between **Collapse** (`display: none`) to remove an element from document flow before first paint, or **Preserve Space** (`visibility: hidden`) to keep rigid CSS Grid and Flexbox layouts from shifting.
- **Granular Site and Path Scoping:** Apply rules to a specific domain, automatically match subdomains, target all websites with `*`, or restrict rules to specific URL paths (including wildcard paths such as `/users/*/settings`).
- **SPA Navigation Support:** Tracks client-side route changes via `history.pushState` and updates active styles automatically without needing a full page reload.
- **Rule Management and Portability:** Toggle the extension globally, enable or disable individual rules per site, and export or import your configuration as JSON.
- **Zero Dependencies and Fully Local:** Built with standard JavaScript and Node.js built-ins. No analytics, no telemetry, and no external network requests.

## How It Is Built

Refloow-Eleblocker is built on Manifest V3 using plain JavaScript with a lightweight Node.js build and test pipeline. To guarantee elements never flash on screen during page load, it uses a dual-path injection strategy:

1. **Navigation-Commit CSS Injection (`src/background/service-worker.js`)**  
   The background service worker listens to `chrome.webNavigation.onCommitted` and compiles matching rules for the target URL into a stylesheet. It injects the CSS via `chrome.scripting.insertCSS` using `origin: "USER"`. User-origin stylesheets bypass strict page Content Security Policies (CSP) and take precedence over author styles, preventing blocked elements from ever painting.

2. **Synchronous `document_start` Cache (`src/content/blocker.js`)**  
   Manifest V3 service workers can sleep between navigations. To prevent any delay while the worker wakes up, a lightweight content script runs at `document_start`—before `<head>` or `<body>` is parsed—and reads a per-host rule cache from `localStorage` to apply styles synchronously. As soon as `chrome.storage.local` resolves, the cache and active stylesheet are synchronized.

3. **Isolated Element Picker (`src/content/picker.js`)**  
   When activated from the popup, the visual picker mounts its highlight box and status bar inside a closed Shadow DOM so the website's own stylesheets cannot interfere with the overlay. It prioritizes stable attributes (`id`, `data-testid`, `data-test`) before falling back to compact structural selectors.

### Project Structure

```text
Refloow-Eleblocker/
├── package.json
├── scripts/
│   └── build.mjs              # Builds Chromium and Firefox MV3 bundles
├── test/
│   └── matcher.test.mjs       # Unit tests using Node's native test runner
└── src/
    ├── manifest.json          # Extension manifest (MV3)
    ├── shared/
    │   ├── matcher.js         # Host/path matching, selector validation, CSS generation
    │   └── storage.js         # chrome.storage.local helpers
    ├── background/
    │   └── service-worker.js  # Pre-paint USER-origin CSS injection and SPA sync
    ├── content/
    │   ├── blocker.js         # Synchronous document_start stylesheet fallback
    │   └── picker.js          # Shadow DOM interactive element picker
    └── popup/
        ├── popup.html         # Extension popup UI
        ├── popup.css
        └── popup.js           # Rule CRUD, validation, JSON import/export
```

## Installation

### Prerequisites

- [Node.js](https://nodejs.org/) v18.0.0 or higher

### Build from Source

Clone the repository and run the build script for your browser:

```bash
git clone [https://github.com/Refloow/Refloow-Eleblocker.git](https://github.com/Refloow/Refloow-Eleblocker.git)
cd Refloow-Eleblocker
```

For **Chrome, Edge, Brave, or other Chromium browsers**:

```bash
npm run build
```

For **Firefox (v115+)**:

```bash
npm run build:firefox
```

### Load in Your Browser

- **Chromium browsers:**
  1. Navigate to `chrome://extensions` (or your browser's equivalent).
  2. Enable **Developer mode**.
  3. Click **Load unpacked** and select the `dist/chromium` directory.

- **Firefox:**
  1. Navigate to `about:debugging#/runtime/this-firefox`.
  2. Click **Load Temporary Add-on...** and select `dist/firefox/manifest.json`.

## Usage

### Blocking an Element Visually

1. Open the website containing the element you want to remove.
2. Click the **Refloow-Eleblocker** icon in your browser toolbar.
3. Click **Pick Element on Page**.
4. Hover over the unwanted element until it is highlighted, then click to block it (press `Esc` to cancel).

### Adding Rules Manually

Open the popup and enter:

- **Website:** A domain such as `example.com` (matches both `example.com` and subdomains like `www.example.com`) or `*` to apply across all websites.
- **Path (optional):** A path prefix such as `/feed`, or a wildcard pattern like `/users/*/settings`. Leave blank to apply to the entire domain.
- **CSS Selector:** Any valid CSS selector (for example, `.cookie-banner` or `#promo-header`).
- **Mode:** Choose `Collapse` to remove the element completely from layout, or `Preserve space` to make it invisible while keeping its dimensions intact.

### Importing and Exporting Rules

Use the **Export JSON** and **Import JSON** buttons at the bottom of the popup to back up your rules or move them between browsers.

## Running Tests

The matching and CSS compilation logic is tested using Node's built-in test runner with zero external test frameworks:

```bash
npm test
```
