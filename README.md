# GEPER corporate website

Static corporate/informational site for GEPER. Astro 7 · Tailwind v4 (compiled) · Three.js (npm, lazy-loaded on the home page only).

## Commands

```
npm install
npm run dev        # local dev server
npm run build      # production build → dist/, then scripts/verify-dist.mjs (fails on Stitch remnants, forbidden claims, broken links)
npm run preview    # serve dist/
```

Deploy `dist/` to any static host. Routes are `/technology`, `/research`, `/platform`, `/about`, `/contact`, `/privacy`, `/terms`; the host should serve `404.html` for unknown paths.

## Environment variables (`.env`, see `.env.example`)

| Variable | Required | Purpose |
|---|---|---|
| `PUBLIC_SITE_URL` | recommended | Canonical origin for `<link rel=canonical>` and Open Graph URLs |
| `PUBLIC_CONTACT_EMAIL` | recommended | Email shown on Contact and used by the mailto fallback |
| `PUBLIC_CONTACT_ENDPOINT` | optional | HTTPS endpoint for the contact form. Empty → form opens the visitor's email client instead (and says so) |
| `PUBLIC_ANALYTICS_SRC` | optional | Script URL injected in `<head>`. Empty → no analytics |

Values are read at build time.

## Contact endpoint contract (backend is NOT included)

`POST {PUBLIC_CONTACT_ENDPOINT}` with `Content-Type: application/json`:

```json
{ "name": "…", "email": "…", "organization": "… (optional)", "inquiryType": "research|platform|data|press|general",
  "message": "…", "website": "", "submittedAt": "ISO-8601" }
```
`website` is a honeypot; reject any request where it is non-empty.

Responses the frontend handles:

| Status | Body | Frontend behaviour |
|---|---|---|
| `202` (or `200`) | `{ "id": "…" }` optional | success state, form cleared, reference shown |
| `400` | `{ "errors": { "email": "msg", … } }` | field errors shown inline |
| `429` | — | "try again later" + mailto fallback link |
| other / network / 15 s timeout | — | error state + mailto fallback link carrying the drafted message |

The endpoint owns rate limiting, spam filtering and email relay. No authentication is used or implied.

## Content

All copy lives in `src/data/content.ts` and `src/data/site.ts`. Status badges accept only
`in-development | planned | active | out-of-scope`. `research.publications`, `technicalNotes` and `validation`
are empty arrays that render honest empty states — add entries only for public material.

## Structure

```
src/
  layouts/BaseLayout.astro            head, skip link, header, footer
  components/nav       Header, MobileMenu (details-based, JS-enhanced), Footer
  components/ui        Button, Eyebrow, Icon (inline SVG set), StatusBadge, SectionHeader, NeedsContent
  components/system    PipelineBackbone, ArchitectureDiagram (static SVG)
  components/cards     Card, CapabilityCard, PrincipleCard
  components/sections  Section, NextStep
  components/hero      DnaNarrative.astro (fallback SVG + controller), dna-scene.ts (Three.js)
  components/forms     ContactForm.astro
  data/                site.ts (identity, nav, backbone), content.ts (page copy)
  styles/global.css    design tokens (@theme) and base styles
  pages/               index, technology, research, platform, about, contact, privacy, terms, 404
scripts/verify-dist.mjs  post-build guard
public/brand, public/favicon*.png, public/apple-touch-icon.png
```
