# PEACHKO — Developer & AI Assistant Guide

## Project Overview

PEACHKO is a Seoul-based K-Beauty on-demand makeup booking service. This is a **static website** built with vanilla HTML5, CSS3, and ES6 JavaScript — no build tools, no package manager, no compilation step.

**Live features:**
- Marketing landing page with dynamic content sections
- Artist booking with Firebase Firestore slot-availability checking
- Firebase Authentication (login/signup)
- Multilingual UI: English, Korean, Japanese, Chinese

---

## Repository Structure

```
pichiko-website/
├── index.html       Landing page (hero, services, artists, reviews, booking CTA)
├── booking.html     Booking form with Firestore real-time slot availability
├── login.html       Firebase Auth login/signup with language sync
├── privacy.html     Privacy policy with language tabs
├── script.js        Firebase init, i18n engine, all render functions (391 lines)
└── style.css        Complete design system via CSS custom properties (664 lines)
```

There is no `src/`, no `node_modules/`, no `package.json`. All files live flat in the root.

---

## Development Workflow

**No build step required.** Serve the root directory with any HTTP server:

```bash
python3 -m http.server 8080
# or: npx serve .
# or: VS Code Live Server extension
```

Then open `http://localhost:8080` in a browser. Firebase SDK and fonts load from CDN on demand — an internet connection is required.

**Testing changes:** Edit the HTML/CSS/JS files and reload the browser. No hot-reload, no bundler watch mode.

---

## External Dependencies (all CDN)

| Dependency | Version | How loaded |
|---|---|---|
| Firebase App | 12.12.1 | ES module from `gstatic.com` |
| Firebase Auth | 12.12.1 | ES module from `gstatic.com` |
| Firebase Firestore | 12.12.1 | ES module from `gstatic.com` (booking.html only) |
| Cormorant Garamond | @4 | `<link>` from jsDelivr CDN |
| DM Sans | @4 | `<link>` from jsDelivr CDN |

The Firebase project is `peachko-356d4` (auth domain: `peachko-356d4.firebaseapp.com`). Config is hardcoded in `script.js` and `booking.html`.

---

## Architecture & Key Systems

### i18n — Multilingual System

All UI strings live in the `T` constant at the top of `script.js` (and `BOOK_T` in `booking.html`):

```js
const T = {
  EN: { nav0: "Services", hero_h1a: "K-Beauty,", ... },
  KO: { nav0: "서비스",   hero_h1a: "K-뷰티,",  ... },
  JP: { ... },
  ZH: { ... }
};
```

DOM elements use `data-key` attributes to map to translation keys:

```html
<button data-key="nav0">Services</button>
```

`applyTranslations(t)` iterates all `[data-key]` elements and sets `textContent`. `render(lang)` is the top-level entry point and calls all section renderers plus `applyTranslations`.

**When adding any user-visible text:** add the string to all four language keys (`EN`, `KO`, `JP`, `ZH`) in `T`. Never hardcode display text directly in HTML unless it does not need translation.

### Data-driven Rendering

Several sections are not static HTML — they are rendered from JS template functions:

| Function | Target element | Data source |
|---|---|---|
| `renderProbLists(t)` | `#prob-bad-list`, `#prob-good-list` | `T[lang].prob_*` |
| `renderSteps(t)` | `#steps-grid` | `T[lang].step_*` |
| `renderServices(t)` | `#services-grid` | `T[lang].svc_*` |
| `renderArtists(t)` | `#artists-grid` | `T[lang].art_*` + `ARTIST_PHOTOS` |
| `renderReviews(t)` | `#reviews-grid` | `T[lang].rev_*` |

To add a new card/item to any of these sections, update the data in all four `T` entries and update the corresponding render function.

**Artist photos** are stored as a base64 array `ARTIST_PHOTOS` in `script.js` (index 0 = Soyeon, 1 = Minjae, 2 = Haerin).

### Routing & State

This is a **multi-page site** — each HTML file is a distinct page. Navigation uses:
- `data-scroll="sectionId"` attributes for in-page smooth scroll
- `onclick="location.href='login.html'"` for page transitions
- URL query parameters to pass state between pages

**URL parameters:**
- `?lang=EN|KO|JP|ZH` — active language (read on every page load)
- `?artist=soyeon|minjae|haerin` — pre-selects artist on booking.html

`currentLang` is read from `new URLSearchParams(window.location.search).get("lang")` on each page.

### Firebase Firestore (booking.html)

`loadBookedSlots(artist, date)` queries Firestore for existing reservations and highlights conflicts in the UI. On form submit, a new document is written to the `reservations` collection with fields: `artist`, `date`, `reservationTime`, `email`, and others.

Form field IDs follow the prefix pattern `f-*`: `f-service`, `f-artist`, `f-date`, `f-time`.

### Firebase Auth (script.js + login.html)

`onAuthStateChanged(auth, user => updateAuthButton(user))` updates the header login/logout button on every page. `signOut(auth)` is called from the logout button.

---

## Naming Conventions

| Scope | Convention | Examples |
|---|---|---|
| CSS classes | kebab-case | `.artist-card`, `.btn-accent`, `.prob-grid` |
| CSS IDs | kebab-case | `#langToggle`, `#artists-grid`, `#floatChat` |
| JS variables/functions | camelCase | `currentLang`, `initBooking()`, `renderArtists()` |
| JS module-level constants | UPPER_SNAKE_CASE | `T`, `ARTIST_PHOTOS`, `LANG_LABELS`, `BOOK_T` |
| CSS custom properties | `--kebab-case` | `--accent`, `--fd`, `--shadow-lg` |
| Form fields | `f-` prefix | `f-service`, `f-artist`, `f-date`, `f-time` |
| Section labels | `lbl-` prefix | `lbl-service` |

---

## Design System

All visual tokens are CSS custom properties defined in the `:root` block at the top of `style.css`. **Always use these tokens — never hardcode colors, fonts, or shadows.**

### Color Palette

| Token | Value | Use |
|---|---|---|
| `--bg` | `#FEF9F5` | Page background |
| `--cream` | `#FFFCF9` | Cards, elevated surfaces |
| `--peach` | `#FFE4D2` | Section backgrounds |
| `--peach-mid` | `#FFAF8A` | Scrollbar thumb, accents |
| `--accent` | `#E16642` | Primary CTA color (coral) |
| `--accent-dk` | `#C04E2C` | Hover/pressed accent |
| `--accent-lt` | `#F08060` | Light accent variant |
| `--text` | `#241208` | Primary text (dark brown) |
| `--muted` | `#846252` | Secondary / caption text |
| `--muted-lt` | `#B08878` | Placeholder text |
| `--border` | `#E8D0C0` | Borders, dividers |
| `--gold` | `#BC9464` | Luxury accent |
| `--gold-lt` | `#D4B890` | Light gold variant |

### Typography

| Token | Value | Use |
|---|---|---|
| `--fd` | `'Cormorant Garamond', serif` | Display / headings |
| `--fb` | `'DM Sans', sans-serif` | Body / UI text |

Font weights in use: Cormorant Garamond 400, 400-italic, 600 · DM Sans 300, 400, 500.

Utility classes: `.fd` (apply serif), `.fb` (apply sans-serif), `.accent` (apply `--accent` color).

### Shadows & Radii

```css
--shadow-sm:  0 2px 8px rgba(200,90,50,.07)
--shadow-md:  0 6px 24px rgba(200,90,50,.10)
--shadow-lg:  0 16px 48px rgba(200,90,50,.13)
--shadow-xl:  0 24px 72px rgba(200,90,50,.16)

--r-sm: 2px   /* buttons, badges */
--r-md: 4px   /* cards */
```

### Easing

```css
--ease:      cubic-bezier(.22,.68,0,1.2)   /* springy */
--ease-soft: cubic-bezier(.4,0,.2,1)       /* material standard */
```

### Responsive Breakpoints

```css
@media (max-width: 900px)  { /* tablets, grid collapse */ }
@media (max-width: 768px)  { /* mobile landscape */       }
@media (max-width: 480px)  { /* mobile portrait */        }
```

Use `clamp()` for fluid typography (e.g. `font-size: clamp(26px, 3.2vw, 46px)`).  
Use CSS Grid `auto-fit / minmax()` for inherently responsive card grids — avoid extra breakpoints where possible.

Do not introduce utility class patterns (no Tailwind-style `.p-4`, `.text-lg`). Keep all selectors semantic.

---

## Adding a New Page

1. Copy the `<head>` block from `index.html` (font links + `style.css` reference).
2. Add `<script type="module">` at the bottom — **`type="module"` is required** for ES module imports.
3. Add a `BOOK_T`-style translation constant in the inline script covering all four languages.
4. Read `?lang=` from URL params and call your `applyLang()` equivalent on load.
5. If the page needs Firebase Auth state, import `getAuth` and `onAuthStateChanged`.

---

## Deployment

Deploy the root directory as-is to any static host. No build command.

```
Vercel:    publish directory = .  (root), no build command
Netlify:   publish directory = .  (root), no build command
GitHub Pages: publish root or /docs
```

---

## What Not To Do

- **Do not introduce a build system** (webpack, Vite, etc.) without explicit discussion — it would require migrating all CDN imports.
- **Do not add TypeScript** without explicit discussion.
- **Do not hardcode colors, fonts, or shadows** — use the CSS custom properties.
- **Do not add a new page** without covering all four languages (EN/KO/JP/ZH).
- **Do not create utility-class CSS** — all selectors should be semantic.
- **Do not use `innerHTML` for user-supplied data** — construct DOM nodes or sanitize inputs to avoid XSS.
- **Do not add a `.gitignore` that excludes HTML/CSS/JS** — all source files are committed directly.
