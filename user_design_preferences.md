---
name: Design preferences — website aesthetic + structure
description: Jeffrey's preferred visual design system for personal/portfolio websites — dark aesthetic, centered container, muted text hierarchy, CSS custom properties
type: user
---

## Visual Aesthetic

Muted dark-first design. No hero elements, no loud typography, no gradients. Everything subdued and compact.

- **Background:** `#0d0d0d`
- **Container bg:** `#141414`
- **Border:** `#2a2a2a`
- **Dividers:** `#262626`
- **Text primary:** `#c8c8c8`
- **Text secondary:** `#999`
- **Text muted:** `#808080`

Light mode equivalents (warm off-white):

- bg `#f2f2f0`, container `#ebebea`, border `#d0d0ce`, divider `#dededc`
- text primary `#333`, secondary `#666`, muted `#595959`

## Theme System

- 3-state toggle: dark → light → auto (system-follow)
- Persisted via `localStorage`
- Applied via `data-theme` attribute on `<html>`
- Auto mode: `@media (prefers-color-scheme: light) { :root:not([data-theme]) { ... } }`
- Toggle button: top-right of container, small (12px), letter-spaced, `◑ auto` label
- Smooth 0.2s transitions on bg/color

## Layout

- **Single centered column**, 480px wide, `max-width: 92vw` — for simple/portfolio/info sites
- Container floats centered on page (`display: flex; align-items: center; justify-content: center; min-height: 100vh` on body)
- Container: `border: 1px solid var(--border)`, `padding: 22px 24px`, no border-radius
- No sidebar, no grid layout — pure vertical stack
- **Exception — app/dashboard UIs:** use full laptop/desktop width (no max-width constraint). Multi-panel layouts (sidebar + content) are appropriate when the UI requires multiple views or navigation. The narrow centered column is for simple sites, not apps.

## Typography

- `system-ui, -apple-system, sans-serif` — no Google Fonts
- Uniform small sizing: 11–15px range
- Section labels: 11px, `letter-spacing: 2px`, uppercase, `var(--text-muted)`
- Body text: 13–14px, `line-height: 1.6–1.7`, `var(--text-secondary)`
- Name/headings: 15px, `letter-spacing: 1px`, `font-weight: 400` (no bold)
- No hero text, no large font sizes

## Sections + Structure

- Sections separated by `border-top: 1px solid var(--divider)`
- Each section: `padding: 14px 0`
- Pattern: label → body → optional link (`→ more`, `→ see work`)
- Section links: 12px, muted color, inline, `→` prefix
- Links inherit color, no underline by default; hover lightens

## Links + Interactions

- `a { color: inherit; text-decoration: none; }`
- Hover: bump from muted → secondary color
- Focus-visible: `outline: 1px solid var(--text-muted); outline-offset: 2–3px`
- No underlines, no button-style links

## Navigation

- Back nav: small inline link, `← back` style, top of sub-pages
- No nav bar — use in-page links

## CSS Architecture

- CSS custom properties for all color tokens — single source of truth
- Plain CSS, no preprocessor
- Box-sizing reset: `*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }`

## General Principles

- No framework, no build step for simple sites (plain HTML/CSS/JS preferred unless scale demands otherwise)
- No hero sections, no large images, no flashy animations
- Inspired by: mannucci.me — minimal, text-first, vertical sections
- Desktop-first for anything interactive; `display: none` below 768px for complex features

## Related

[[design-system]] [[css-custom-properties]] [[system-ui]]
