# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

This is a single self-contained static HTML file with no build, test, or lint tooling and no dependencies. There is no `package.json`.

- **Run/preview**: open `prototype-mock.html` directly in a browser (double-click, or `open prototype-mock.html` on macOS).
- **No build step**: all HTML, CSS, and JS live inline in the one file. Edit it directly and reload the browser to see changes.

## Architecture

`prototype-mock.html` renders a clickable device-frame prototype (an iPhone/iPad mock) centered on a black stage, driven by a single in-memory `state` object (`device`, `orientation`, `lang`, `step`, `apiKey`) inside an IIFE at the bottom of the file. There is no framework, router, or persistence (including the API key — it lives only in memory and is lost on reload) — everything is plain DOM manipulation and re-rendered via one `render()` function whenever state changes.

Key structural pieces, top to bottom in the file:

- **`.topbar` controls**: steps-menu toggle, settings toggle, orientation rotate, and restart — each wired to a click handler that mutates `state` and calls `render()`. Device and language are no longer top-level buttons; they live inside the settings popover.
- **`.device` / `.device-screen`**: the visual phone/tablet frame. Switching `state.device` toggles the `iphone`/`ipad` class on `#device`, and `state.orientation` toggles a `landscape` class — CSS handles all of it (frame width/height, corner radius, notch shape/position, home-indicator position) via combined class selectors like `.device.iphone.landscape` — no JS-side layout logic beyond the class swaps. The screen background is white; placeholder content and nav arrows use dark ink for contrast.
- **`.screen-content`**: placeholder per-step content (currently just a step badge/number and label) rendered inside the device screen. Extend this if real per-step mockup content is added.
- **In-frame nav arrows (`#prevBtn`/`#nextBtn`)** and **`.bottombar` progress dots**: both call the shared `goToStep(n)`, which clamps to `1..TOTAL_STEPS` and re-renders.
- **Popovers (`.menu-overlay` / `.menu-panel`)**: a shared pattern used by three anchored popovers — the steps menu (`#menuOverlay`), settings (`#settingsOverlay`), and the API key entry popup (`#apiKeyOverlay`). Each is a transparent full-screen click-catcher with a small anchored panel inside; clicking outside the panel (or `Escape`) closes it. `openMenu()`/`openSettings()` close each other so only one top-level popover is open at a time; `closeSettings()` also closes the nested API key popover.
- **Settings popover**: built by `buildSettingsPanel()`, rebuilt on every `render()`. Contains a language segmented control (`#langSegmented`, EN/DE/FR), a device segmented control (`#deviceSegmented`, iPhone/iPad), and the API key row (`#apiKeyRow`). The API key row shows either an "Add key" button (no key set) or a masked chip (`maskKey()` — fixed 8 dots + last 4 characters) with a remove (✕) button.
- **API key entry popover (`#apiKeyOverlay`)**: opened via `openApiKeyPopover(anchorEl)`, which positions `#apiKeyPanel` under the triggering button using `getBoundingClientRect()` (not a fixed offset like the other popovers, since it can be triggered from a button inside another popover). `Enter` in the input or the Save button commits via `saveApiKey()`; empty input is ignored.
- **i18n**: the `i18n` object holds `en`/`de`/`fr` string tables (including a `step(n)` function for the numbered label). `t()` returns the active language's table. `applyStaticI18n()` generically walks every `[data-i18n]` element and sets `textContent` from the matching key (and `[data-i18n-placeholder]` for input placeholders) — add a new string by giving the element a `data-i18n="key"` attribute and adding that key to all three language tables, no manual wiring needed in `render()`. Dynamically-built elements (segmented control buttons, the API key add/remove button) are labeled directly from `t()` in their build functions instead, since they don't exist in the static markup.

## Conventions to preserve when editing

- `TOTAL_STEPS` is the single source of truth for step count — dots, menu items, and nav-arrow disabled states all derive from it. Change step count there, not by editing markup.
- `LANGS` (`["en", "de", "fr"]`) is the single source of truth for available languages — the language segmented control is built from it. Adding a language means adding it to `LANGS` and adding a matching key to `i18n` with the same shape as the existing entries.
- Adding a static UI string: give the element `data-i18n="key"` in markup and add that key (as a string, not a function) to **all** language tables — `applyStaticI18n()` picks it up automatically. Strings needed only in JS (e.g. dynamically-built buttons) are read via `t().key` directly in the relevant build function.
- Device-frame visual differences (notch vs. camera dot, corner radius, bezel padding, portrait vs. landscape dimensions) are expressed purely in CSS via the `.iphone`/`.ipad` and `.landscape` classes on `#device` — don't branch on `state.device`/`state.orientation` in JS beyond the class swap.
- Disabled affordances (e.g. Restart on step 1, nav arrows at the ends) use the native `disabled` attribute / a `.disabled` class plus `pointer-events: none`, set from `render()` — keep new disabled states consistent with this pattern rather than removing elements from the DOM.
- New popovers should follow the existing `.menu-overlay`/`.menu-panel` pattern (transparent click-catcher + anchored panel, open/close toggled via a class, outside-click and `Escape` dismiss) rather than introducing a different modal style.
