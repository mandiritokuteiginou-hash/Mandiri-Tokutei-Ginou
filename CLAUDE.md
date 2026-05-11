# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static single-page marketing website for **Mandiri Tokutei Ginou** (`mandiritokuteiginou.com`), an Indonesian platform that helps workers apply for Japan's Specified Skilled Worker (SSW / Tokutei Ginou) program without brokers. The entire site is a single `index.html` file with all CSS and JavaScript inlined — there is no build step, no framework, and no package manager.

## Deployment

The site is deployed via **GitHub Pages** with a custom domain configured in `CNAME`. To preview changes locally, open `index.html` directly in a browser or use any static file server:

```bash
python3 -m http.server 8000
# then open http://localhost:8000
```

There are no lint, test, or build commands.

## Architecture

### File structure

```
index.html                  — The entire website (1682 lines)
FINAL-100-completion-report.html  — Internal dev progress report (not served in production)
CNAME                       — GitHub Pages custom domain
```

### `index.html` layout

The file is structured in this order:
1. `<head>` — SEO meta tags, two JSON-LD structured data blocks (Organization + FAQPage), analytics snippet, Google Fonts, and all inline `<style>` CSS
2. HTML body sections (in order): skip link → navbar → `#hero` → testimonials → `#why` (why choose us) → `#cara-kerja` (how it works, 6 steps) → `#sektor` (14 SSW sectors + modal) → `#syarat` (requirements) → `#faq` (accordion) → `#daftar` (quiz funnel + registration form) → sticky CTA bar → floating WhatsApp button
3. A single `<script>` block at the bottom containing all JavaScript

### CSS design tokens

All colors and spacing use CSS custom properties defined in `:root`:

| Variable | Value | Usage |
|---|---|---|
| `--red` | `#C8102E` | Primary brand color |
| `--red-dark` | `#a00d24` | Hover states |
| `--red-light` | `#f5e6e9` | Backgrounds/chips |
| `--gold` | `#C9A84C` | Accent (How it works section) |
| `--off-white` | `#FAF9F7` | Section backgrounds |
| `--ink` / `--ink-mid` / `--ink-soft` | `#111` / `#444` / `#888` | Text hierarchy |

Fonts: **Plus Jakarta Sans** (body) and **Noto Serif JP** (Japanese kanji decorative elements).

### Key JavaScript functions

| Function | Purpose |
|---|---|
| `mtgTrack(event, data)` | Analytics wrapper — sends to GA4 if configured, always logs to console |
| `quizAnswer(step, val)` | Drives the 3-step quiz that segments users before showing the form |
| `showQuizResult()` | Maps quiz answers to one of three segments: `alumni`, `career-switcher`, `fresh-graduate` |
| `handleSubmit(e)` | POSTs form data to the n8n webhook; falls back to a pre-filled WhatsApp link on error |
| `openSector(el)` / `closeSector()` | Populates and shows/hides the sector detail modal |
| `toggleLang()` | Swaps visible text between Indonesian and Japanese (partial translation) |
| `toggleMobileMenu()` | Hamburger nav for mobile |

### External integrations

- **n8n webhook** (form submission): `https://n8n-lb9wcjvt8ui7.emas.sumopod.my.id/webhook/mandiri-tg-daftar` — receives JSON with fields `nama`, `noHp`, `usia`, `pendidikan`, `sektor`, `bahasa`, `source`, `timestamp`, `utm`, `segment`
- **WhatsApp fallback**: `+62-807-418-1493` — used when the webhook fails and throughout the page as direct CTA links
- **GA4**: Placeholder `G-XXXXXXXXXX` in a commented-out script block; `mtgTrack()` is already wired up and will fire events automatically once a real measurement ID is uncommented

### Interactive patterns

- **Scroll reveal**: Elements with class `reveal` animate in via `IntersectionObserver`
- **Lazy rendering**: `.sectors`, `.reqs`, `.faq`, `.cta-section` use `content-visibility: auto` for performance
- **Sticky CTA bar**: Appears after 600px scroll, hides when the `#daftar` form section is in the viewport
- **Quiz → form flow**: The quiz card (`#quizCard`) hides and the form card (`#formCardMain`) shows after quiz completion; `quizData.segment` is appended to the form payload
- **Sector modal**: Sector data (`name`, `jp`, `salary`, `demand`, `desc`, `skills`) is stored as `data-*` attributes on each `.sector-chip` element and read into the modal by `openSector()`

## Conventions

- All section anchor IDs are in Indonesian: `#cara-kerja`, `#sektor`, `#syarat`, `#faq`, `#daftar`
- WhatsApp message text is URL-encoded Indonesian
- Phone numbers use Indonesian mobile format: `08xxxxxxxxxx` (validated with `/^08[0-9]{8,12}$/`)
- User-facing copy is Indonesian; partial Japanese translation exists only for hero heading, subheading, section titles, CTA text, and the form title
- Sector skills are pipe-delimited strings in `data-skills` attributes (`skill1|skill2|skill3`)
- Input sanitization strips HTML tags and `<>"'&` characters, then truncates to 200 chars
