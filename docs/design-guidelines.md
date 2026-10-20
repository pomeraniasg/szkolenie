# Design Guidelines — x-kom.pl reference

Source: https://www.x-kom.pl, captured 2026-10-20 with Playwright (cookie modal accepted, promo header left in place).
This is a **reference for matching a brand**, not an agreed design for the AI Chat app. Values marked *not measured* were not extracted from the site.

## Assets

| Asset | Path (relative to repo root) | Notes |
|---|---|---|
| Full-page homepage screenshot | [`assets/homepage.png`](../assets/homepage.png) | Cookie modal dismissed before capture |
| Logo (wordmark, inline SVG from header) | [`assets/logo.svg`](../assets/logo.svg) | Colors come from CSS variables; defaults to `#000` |
| Favicon | [`assets/favicon.ico`](../assets/favicon.ico) | Served from `assets.x-kom.pl` |
| Font: Instrument Sans (variable, wdth + wght) | [`assets/fonts/instrument-sans/InstrumentSans-VariableFont_wdth,wght.woff2`](../assets/fonts/instrument-sans/InstrumentSans-VariableFont_wdth,wght.woff2) | Self-hosted on the site |
| Design tokens (JSON) | [`assets/design-tokens.json`](../assets/design-tokens.json) | Machine-readable version of this document |

> **License check required:** the logo and the Instrument Sans font are the property of x-kom / their font licensor. Verify usage rights before using or redistributing them in the app.

## Colors

| Token | Hex | Usage |
|---|---|---|
| `brand.primary` | `#222222` | Primary buttons ("Kup teraz"), dark blocks |
| `brand.accent` | `#0082FA` | Links / highlights (observed as a link-blue token on the page) |
| `brand.success` | `#007D00` | Positive labels, e.g. discount tags |
| `brand.error` | — | *Not measured* |
| `background.default` | `#FFFFFF` | Page and header background |
| `background.light` | `#F7F7F7` | Subtle section backgrounds |
| `background.overlay` | `rgba(0, 0, 0, 0.08)` | Dim/overlay tint |
| `text.primary` | `#1A1A1A` | Body and headings |
| `text.secondary` | `#444444` | Supporting text |
| `text.muted` | `#6E6E6E` | Captions, meta information |
| `text.onDark` | `#FFFFFF` | Text on `#222222` buttons and dark blocks |
| `extra.promo` | `#B50067` | Promotional text accents |
| `extra.successSurface` | `#EEF7ED` | Background of success tags |
| `extra.border` | `#DDDDDD` | Dividers and borders |

## Typography

- **Font family:** `"Instrument Sans", sans-serif` — self-hosted variable font, `font-display: swap`, weight range 400–700.
- **Weights:** regular `400`, semibold `600`, bold `700`. Medium `500` was not observed.
- **Sizes:** `13px` (sm), `14px` (base), `16px` (md), `20px` (lg), `26px` (xl, h2), `28px` (display, h1).
- **Line height:** `1.2` tight, `1.4` base, `1.6` relaxed (scale not measured; `normal` and `32px` were observed on headings).

## Spacing

Base unit: **4px**. Scale in `design-tokens.json`: `4, 8, 12, 16, 20, 24, 28, 32px`.
*Not measured* from the site: the scale is the default from the skill template. Check real section gaps in the browser before relying on it.

## Border radius

| Token | Value | Usage |
|---|---|---|
| `xs` | 2px | Small inline elements |
| `sm` | 4px | Tags and discount labels |
| `md` | 8px | Buttons |
| `lg` | 12px | Cards and larger containers |
| `full` | 512px | Pill-shaped elements |
| `circle` | 50% | Avatars and round icons |

## Components

- **Header:** white background, 72px tall, logo on the left, search and account/cart icons on the right. A promo bar sits above it (dark text on a light strip, e.g. "Trwa Black Weeks…").
- **Navigation:** text links in `#1A1A1A`, 14px, below the header.
- **Primary button:** `#222222` background, `#FFFFFF` text, `12px 24px` padding, `8px` radius, 16px text.
- **Tag / discount label:** `#EEF7ED` background, `#007D00` text, `4px` radius, 13px text.
- **Inputs:** *not measured*.
- **Cookie consent modal:** overlay with a centered dialog; the site offers "Akceptuję wszystkie", "Ustawienia cookies" and "Odrzucam niewymagane". Not a component to copy.

## Logo usage

- Use the wordmark on white or light backgrounds with its default dark fill (`#000` or `#222222`).
- Keep the clear space at least the height of the "x" in the wordmark around it.
- Do not stretch the SVG; keep its `118 × 32` aspect ratio.
- An inverted (white) variant was **not** found in the site's assets. Derive one only if the brand owner approves it.

## Visual style summary

The site feels like a large, practical electronics retailer: clean white surfaces, dense product grids and a restrained near-black primary color. Blue is used sparingly for links and highlights, and green and magenta mark discounts and promotions. Typography is one geometric-grotesque family (Instrument Sans) with clear hierarchy from 14px body text to 28px headings. Rounded corners are modest (8px buttons), and shadows and gradients are light, so the interface reads as functional rather than decorative.
