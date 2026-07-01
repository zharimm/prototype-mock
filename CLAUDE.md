# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

This is a single self-contained static HTML file with no build, test, or lint tooling and no dependencies. There is no `package.json`.

- **Run/preview**: open `prototype-mock.html` directly in a browser (double-click, or `open prototype-mock.html` on macOS).
- **No build step**: all HTML, CSS, and JS live inline in the one file. Edit it directly and reload the browser to see changes.

## Architecture

`prototype-mock.html` renders a clickable device-frame prototype (an iPhone/iPad mock) centered on a black stage, driven by a single in-memory `state` object (`device`, `lang`, `step`) inside an IIFE at the bottom of the file. There is no framework, router, or persistence — everything is plain DOM manipulation and re-rendered via one `render()` function whenever state changes.

Key structural pieces, top to bottom in the file:

- **`.topbar` controls**: menu toggle, device switcher (iPhone/iPad), language switcher (EN/DE), and restart — each wired to a click handler that mutates `state` and calls `render()`.
- **`.device` / `.device-screen`**: the visual phone/tablet frame. Switching `state.device` toggles the `iphone`/`ipad` class on `#device`, which CSS handles entirely (frame size, corner radius, notch shape) — no JS-side layout logic. The screen background is white; placeholder content and nav arrows use dark ink for contrast.
- **`.screen-content`**: placeholder per-step content (currently just a step badge/number and label) rendered inside the device screen. Extend this if real per-step mockup content is added.
- **In-frame nav arrows (`#prevBtn`/`#nextBtn`)** and **`.bottombar` progress dots**: both call the shared `goToStep(n)`, which clamps to `1..TOTAL_STEPS` and re-renders.
- **Menu popover (`#menuOverlay` / `#menuPanel`)**: anchored under the hamburger button (not a full-screen modal). Built dynamically in `buildMenu()` from `TOTAL_STEPS`, and closes on outside click, `Escape`, or step selection.
- **i18n**: the `i18n` object holds `en`/`de` string tables (including a `step(n)` function for the numbered label). `t()` returns the active language's table; every label-bearing element is updated from it inside `render()`. Add a language by adding a new key to `i18n` with the same shape.

## Conventions to preserve when editing

- `TOTAL_STEPS` is the single source of truth for step count — dots, menu items, and nav-arrow disabled states all derive from it. Change step count there, not by editing markup.
- Adding a UI string requires adding it to **both** `i18n.en` and `i18n.de` with matching keys, then reading it via `t()` inside `render()` (see existing `restart`, `menuTitle`, `deviceToggleToIpad/Iphone` entries).
- Device-frame visual differences (notch vs. camera dot, corner radius, bezel padding) are expressed purely in CSS via the `.iphone` / `.ipad` class on `#device` — don't branch on `state.device` in JS beyond the class swap.
- Disabled affordances (e.g. Restart on step 1, nav arrows at the ends) use the native `disabled` attribute / a `.disabled` class plus `pointer-events: none`, set from `render()` — keep new disabled states consistent with this pattern rather than removing elements from the DOM.
