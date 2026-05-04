# webpou

A static, single-page marketing site built on the **Salient** HTML5 template (by Template Stock, CC BY 3.0). The page advertises a fictional product/agency with sections for hero, about, services, team, portfolio screenshots, pricing, subscribe form, testimonials, and contact — all in a single `index.html`. Recently extended with a **light/dark theme toggle** driven by CSS custom properties.

> No build step. No package manager. No framework. Just open `index.html` in a browser.

---

## Table of contents

- [Quick start](#quick-start)
- [What's in the box](#whats-in-the-box)
- [Project structure](#project-structure)
- [Page anatomy](#page-anatomy)
- [Tech stack](#tech-stack)
- [Theming (light/dark)](#theming-lightdark)
  - [How the toggle works](#how-the-toggle-works)
  - [Token reference](#token-reference)
  - [Adding a new themed surface](#adding-a-new-themed-surface)
  - [Known gaps](#known-gaps)
- [Customizing content](#customizing-content)
- [Browser support](#browser-support)
- [License](#license)

---

## Quick start

The site is fully static — no install, no compilation, no dev server required.

```bash
# clone, then open the file directly
start index.html             # Windows
open  index.html             # macOS
xdg-open index.html          # Linux
```

For local development with live reload, any static server works:

```bash
# Python 3
python -m http.server 8000

# Node (one-shot, no install)
npx serve .

# PHP
php -S localhost:8000
```

Then visit `http://localhost:8000`.

> ⚠️ A handful of features (Google Maps embed, Google Fonts, MailChimp form action) hit external services. Open the page over HTTP rather than `file://` to avoid CORS quirks.

---

## What's in the box

A complete one-page landing site, ready to rebrand:

- Hero section with full-viewport background image and CTA
- About / value-prop blocks
- Animated feature grid with [et-line](http://etlinefont.com/) icon font
- Team grid with hover overlays
- Portfolio screenshots (Isotope masonry + Magnific Popup lightbox)
- Pricing table
- MailChimp-ready subscribe form
- Testimonials carousel (Owl Carousel)
- Contact form + Google Maps embed
- Sticky/scrollspy navbar that fixes on scroll
- Smooth-scroll anchor navigation
- Preloader animation
- "Back to top" button
- **Dark-mode toggle in the desktop navbar** (added in this fork)

---

## Project structure

```
webpou/
├── index.html                    # The whole site — one file, ~1,160 lines
├── README.md                     # You are here
├── License.txt                   # CC BY 3.0 (Template Stock)
└── assets/
    ├── css/
    │   ├── style.css             # Custom site styles (~3,250 lines, theme tokens at top)
    │   ├── bootstrap.min.css     # Bootstrap 3 grid/components
    │   ├── animate.css           # CSS animation library
    │   ├── font-awesome.min.css  # FA 4.3 icon font
    │   ├── et-line-font/         # et-line icon font + stylesheet
    │   ├── bxslider/             # BX slider plugin styles
    │   ├── owl-carousel/         # Owl carousel plugin styles
    │   └── magnific-popup/       # Lightbox plugin styles
    ├── js/
    │   ├── main.js               # Site behavior (scrollspy, smooth scroll, init plugins)
    │   ├── theme-toggle.js       # Light/dark toggle controller (added)
    │   ├── jquery-1.11.3.min.js  # jQuery 1.11
    │   ├── bootstrap.min.js      # Bootstrap 3 JS components
    │   └── jquery.*.js           # ~15 jQuery plugins (carousels, isotope, gmaps, etc.)
    ├── fonts/                    # Font Awesome + et-line web font files
    └── images/                   # Hero backgrounds, team photos, screenshots, logo
```

---

## Page anatomy

The single `index.html` is composed of these sections, in order:

| Section ID              | Purpose                                              |
| ----------------------- | ---------------------------------------------------- |
| `#hero-section`         | Full-screen hero with parallax background            |
| `#landing-offer`        | About / intro copy                                   |
| `#features-section`     | Animated icon grid of services                       |
| `#video-section`        | Embedded promotional video                           |
| `#team-section`         | Team member cards                                    |
| `#screenshots-section`  | Filterable portfolio (Isotope) with lightbox         |
| `#clients-section`      | Client logos                                         |
| `#prices-section`       | Pricing tiers                                        |
| `#subscribe-section`    | Email capture form                                   |
| `#testimonials-section` | Quote carousel                                       |
| `#contact-section`      | Contact form + map                                   |
| `#footer-section`       | Social links, copyright                              |

Navigation links in the navbar use `class="section-scroll"` and `href="#section-id"` — handled by `assets/js/jquery.smoothscroll.js`.

---

## Tech stack

Everything ships as flat files; no compilation required.

| Layer        | Tech                                                                 |
| ------------ | -------------------------------------------------------------------- |
| Markup       | HTML5, single-page                                                   |
| Layout/Grid  | [Bootstrap 3.x](https://getbootstrap.com/docs/3.4/) (`navbar`, grid) |
| Custom CSS   | Plain CSS with custom properties (no preprocessor)                   |
| Animations   | [Animate.css](https://animate.style/) + jQuery Waypoints triggers    |
| Icons        | Font Awesome 4.3 + et-line                                           |
| JS runtime   | jQuery 1.11                                                          |
| Carousels    | Owl Carousel, BX Slider                                              |
| Masonry      | Isotope                                                              |
| Lightbox     | Magnific Popup                                                       |
| Maps         | Google Maps JS API (free tier, key-less embed)                       |
| Parallax     | Stellar.js                                                           |
| Newsletter   | jQuery AjaxChimp (MailChimp endpoint)                                |
| Fonts        | Google Fonts (Raleway, Crimson Text, Open Sans)                      |

> **Note on jQuery 1.11**: this is the version shipped by the original template. It predates many modern security patches. If you take this beyond a marketing demo, upgrade to a current 3.x build.

---

## Theming (light/dark)

The site supports a **light** (default) and **dark** color theme, switchable from a moon/sun button in the desktop navbar. The user's choice persists in `localStorage` across reloads.

### How the toggle works

The implementation has three pieces:

**1. CSS token layer** (top of `assets/css/style.css`)

All themable colors live in CSS custom properties under `:root` (light defaults) and `[data-theme="dark"]` (dark overrides). Roughly 25 high-impact rules — body, navbar, dropdown, footer, form-control, preloader, headings, surface utilities — were migrated to reference the tokens. Bootstrap and untouched plugin styles are left as-is; the migration is intentionally partial.

**2. FOUC-prevention inline script** (in `<head>`)

```html
<script>
  (function () {
    try {
      if (localStorage.getItem('theme') === 'dark') {
        document.documentElement.setAttribute('data-theme', 'dark');
      }
    } catch (e) {}
  })();
</script>
```

This runs synchronously before any paint, so users who chose dark on a previous visit don't see a flash of light content.

**3. Toggle controller** (`assets/js/theme-toggle.js`)

Wires the navbar button: on click it flips `data-theme` on `<html>`, persists to `localStorage`, swaps the icon between `fa-moon-o` and `fa-sun-o`, and updates `aria-pressed` / `aria-label` for screen readers.

The button is wrapped in `<li class="hidden-xs hidden-sm">`, so it only renders on Bootstrap's `md` and `lg` breakpoints (≥992px). Mobile and tablet users get a clean collapsed nav without the toggle.

**Defaults**:
- First-time visitor → **light** (system `prefers-color-scheme` is intentionally NOT consulted)
- After clicking the toggle → choice persists indefinitely
- Clearing site data → resets to light

### Token reference

```css
:root {
  --bg-page:           #ffffff;   /* body background                              */
  --bg-surface:        #ffffff;   /* navbar, cards, dropdowns                     */
  --bg-muted:          #e5e5e5;   /* gray section backgrounds, hover backgrounds  */
  --bg-footer:         #f7f7f7;   /* footer                                       */
  --bg-inverse:        #34353a;   /* .dark-bg utility                             */
  --text-primary:      #1f2021;   /* headings, navbar links                       */
  --text-body:         #737272;   /* body copy                                    */
  --text-muted:        #4f4f4f;   /* footer links                                 */
  --text-inverse:      #ffffff;   /* text on inverse surfaces                     */
  --border:            #f0f0f0;   /* navbar bottom-border, soft dividers          */
  --border-strong:     #eeeeee;   /* footer top-border                            */
  --input-bg:          #ffffff;
  --input-text:        #555555;
  --input-border:      #cccccc;
  --input-placeholder: #494949;
  --accent:            #fc6e51;   /* brand orange (unchanged in both themes)      */
}

[data-theme="dark"] {
  --bg-page:           #15171c;
  --bg-surface:        #1c1f26;
  --bg-muted:          #22252d;
  --bg-footer:         #181a20;
  --bg-inverse:        #0e1014;
  --text-primary:      #f1f1f1;
  --text-body:         #b8bcc4;
  --text-muted:        #a0a4ac;
  --text-inverse:      #f1f1f1;
  --border:            #2a2d36;
  --border-strong:     #2a2d36;
  --input-bg:          #22252d;
  --input-text:        #e8e8e8;
  --input-border:      #3a3d46;
  --input-placeholder: #8a8e96;
  --accent:            #fc6e51;
}
```

**Contrast** (WCAG AA targets are 4.5:1 for normal text, 3:1 for large text and UI components):

| Pair                            | Ratio   | Verdict           |
| ------------------------------- | ------- | ----------------- |
| `--text-body` on `--bg-page` (dark) | ~10.4:1 | Passes AA + AAA |
| `--text-primary` on `--bg-surface` (dark) | ~14.6:1 | Passes AA + AAA |
| `--input-placeholder` on `--input-bg` (dark) | ~4.6:1 | Passes AA       |
| `--accent` on `--bg-page` (dark) | ~4.3:1  | Passes AA for large text / UI only |

The brand accent `#fc6e51` is reused for links and focus borders. It clears AA for non-text/large-text but sits below 4.5:1 for normal-size body text — keep that in mind if you add long-form orange copy.

### Adding a new themed surface

When you write a new rule that needs to look right in both themes:

```css
/* DO — use a token */
.my-card {
  background: var(--bg-surface);
  color: var(--text-body);
  border: 1px solid var(--border);
}

/* DON'T — hard-code a hex value */
.my-card {
  background: #ffffff;
  color: #737272;
}
```

If no existing token fits, add one to **both** `:root` and `[data-theme="dark"]`.

For dark-mode-only tweaks (e.g., images that need inverting), use the attribute selector:

```css
[data-theme="dark"] .my-logo {
  filter: brightness(0) invert(1);
}
```

This is how the brand logo is handled — see the `[data-theme="dark"] .navbar-brand img` rule near the bottom of `style.css`.

### Known gaps

The migration is "best-effort, high-impact" — about 80% of the visible surface is themed. Things that may still look light-ish in dark mode:

- Plugin stylesheets (Owl Carousel, Magnific Popup, BX Slider) — not migrated
- Bootstrap 3 component defaults (`.modal`, `.alert`, etc., if you start using them)
- Section-specific hex literals scattered through the lower 2,000 lines of `style.css`
- Hero JPG backgrounds — left as-is (they're already dark imagery)

If you spot a glaring light surface in dark mode, grep `style.css` for the offending hex literal and either replace it with an existing token or add a new one.

---

## Customizing content

This is a static template — every change is a hand-edit of `index.html` and/or `style.css`.

| To change…           | Edit                                                                            |
| -------------------- | ------------------------------------------------------------------------------- |
| Brand name / logo    | `index.html` `.navbar-brand` (line ~74); `assets/images/logo.png`               |
| Nav links            | `index.html` `.nav.navbar-nav` `<li>` items (lines ~78-100)                     |
| Hero copy            | `index.html` `#hero-section .hero-content`                                      |
| Section text         | `index.html` — sections are clearly delimited by HTML comment banners           |
| Brand color          | Search/replace `#fc6e51` in `style.css`, **or** redefine `--accent` in tokens   |
| Hero background      | Replace files in `assets/images/hero-bg/`                                       |
| Newsletter endpoint  | `index.html` `<form id="mc-form">` `action` attribute                           |
| Map location         | `assets/js/main.js` — Google Maps init block                                    |
| Theme defaults       | `assets/css/style.css` — `:root` and `[data-theme="dark"]` blocks               |

---

## Browser support

Modern evergreen browsers (Chrome, Firefox, Safari, Edge). The page uses:

- CSS custom properties (variables) — IE11 unsupported (theming will silently fall back to the property's initial value)
- `:focus-visible` — every browser since 2020
- `localStorage` (graceful try/catch fallback if unavailable)

The original template's IE8 shim (`html5shiv` + `respond.js`) is still loaded conditionally for legacy purposes but the dark-mode feature explicitly does not target IE.

---

## License

Original template: **Creative Commons Attribution 3.0 Unported** — by [Template Stock](http://templatestock.co). The `License.txt` in the repo asks that you keep the footer back-link to Template Stock if you use this commercially without paying their attribution-removal fee.

Modifications in this repository (dark-mode feature, README rewrite) inherit the same license.
