# Copilot Instructions for Virtual‑Closet

This repository is a *very simple Chrome extension* built as a student project.  The
primary user experience is a drag‑and‑drop “virtual fitting room” that currently
lives in a static HTML page.  There’s no build system or backend – everything is
plain JavaScript, assets and a Chrome manifest.

## Big picture

1. **Extension manifest (`manifest.json`)**
   * Declares the name, version, permissions and the background script for the
     extension.  This code is still using **manifest_version 2** with a
     non‑persistent background page (see `eventPage.js`).  Changing to v3
     requires rewriting the background script as a service worker and updating
     permissions accordingly.
   * Permissions include `storage`, `activeTab`, `contextMenus` and `tabs`.

2. **Background script (`eventPage.js`)**
   * Runs whenever the extension is active.  It registers a browser action
     (icon click) and a context‑menu item for images.  At the moment it only
     opens the sample page; the context menu has no click handler yet.  To add
     data‑flow when a user chooses **Add item to closet!** you would listen on
     `chrome.contextMenus.onClicked` and write to `chrome.storage`.

3. **UI page (`DressUp_Sample_Website.html`)**
   * This is the page that opens when the icon is clicked.  It contains a set of
     hard‑coded `<div class="draggable">` items with image links under `i/`.
   * CSS is in `styles.css` and a few inline rules enable absolute positioning
     for the draggable items.
   * The drag/drop behaviour is implemented in `drag_drop.js` (simple
     `mousedown`/`mousemove`/`mouseup` handlers and `querySelectorAll`).  If
     you add new DOM elements follow the same pattern; the script needs to be
     re‑run or elements created before it executes.
   * There is commented example code (in both the HTML and JS) showing how the
     bundled **replace‑color** library could be imported as a module to make
     image backgrounds transparent.  These lines are not active but serve as
     documentation for future enhancements.

4. **Libraries (`libraries/` & `libraries/utils/`)**
   * `jimp.js`, `replace-color.js` and several utility validators live here.
     They are copied from external sources; the project does not use npm or any
     package manager.  Inspect `libraries/utils/validate-colors.js` for the
     coding style – ES6 functions, descriptive comments, and input validation.
   * If you need image processing in the extension, include the module via a
     `<script type="module" src="libraries/replace-color.js"></script>` tag
     and follow the commented example in `drag_drop.js`.

## Developer workflows

* **Running / testing** – there is no build step.  
  1. `git clone` the repo.
  2. Open `chrome://extensions/` in Chrome (or a Chromium derivative).
  3. Enable "Developer mode", click "Load unpacked" and point to the project
     root.
  4. Click the extension icon to open `DressUp_Sample_Website.html` and try
     dragging the items or right‑clicking images to see the context menu.
  5. After making JS/HTML changes, hit the reload button for the extension in
     the extensions page; the page itself can be reloaded normally.

* **Editing conventions** – use plain JavaScript, avoid transpilers or bundlers.
  Keep global variables minimal (`item` and `cX` in `drag_drop.js` are
  examples).  Comment intentions clearly; there are already multi‑line comments
  explaining the reason behind certain blocks.

* **Work on manifest** – when adding new permissions or scripts keep the icon
  paths relative and ensure you bump the "version" field so Chrome will
  actually refresh on reload.

* **No automated tests** – everything is exercised manually in the browser.
  If you add complex logic it’s acceptable to create a simple HTML harness or
  a Node script to run it, but there is no Jest/Mocha config in the repo.

## Project‑specific patterns and notes

* The sample page uses `class="draggable"` on wrapper `<div>`s and positions
  them absolutely.  Images inside the div have `pointer-events: none` so that
  mouse events are caught by the wrapper rather than the `<img>` itself.
* `drag_drop.js` arranges the initial items along the top left by incrementing
  `cX` by 40px for each element; if you change the layout update this logic.
* The icon assets (`ext_icon16.png`, etc.) are used in both `manifest.json`
  and for the toolbar button.  Keep their dimensions correct (16, 32, 48, 128).
* Chrome API calls are used directly; no promise wrapper is in place.  You may
  need to add callbacks or migrate to the `chrome.*.promise` style if refactoring.
* Background page is non‑persistent; it only runs when an event fires.  This is
  why there’s no `document` reference in `eventPage.js` – it does not have a DOM.

*Data Flow Summary:* user clicks icon → background script opens local HTML file
→ page loads static assets and JS.  Context menu creation is also triggered by
background but no handler yet; future work should communicate via
`chrome.storage` or by sending messages to content scripts or tabs.

## Useful files to reference

* `manifest.json` – permissions and entry points
* `eventPage.js` – background service logic
* `DressUp_Sample_Website.html` – draggable UI markup + example import
* `drag_drop.js` – the only real business logic at present
* `libraries/utils/validate-colors.js` – style example for small utilities

---

❓ **Next steps for improvement**

- Add brief tests or a script to verify context menu behaviour.
- Implement storage of selected items and retrieval when the user re‑opens
  the sample page.
- Consider migrating to manifest v3 if this repo is kept alive.

Please let me know if anything above is unclear or if you need more details
about a specific part of the codebase!  I'm happy to iterate on these
instructions.