# Interlink — Venue Page Prototype (Handoff)

**File:** `Interlink-VenuePage-Prototype.html`
**Status:** High-fidelity interactive prototype (Screen 1 of 5 from the design brief)
**Date:** June 2026
**Purpose:** Demand-validation / design-partner outreach. Nothing is backed by a server — scheduling, staging, and per-space 3D are mocked. The Overview splat is a *real* SuperSplat embed and is genuinely interactive.

This document is written for the technical partner. It covers what the prototype is, the design system it establishes, how the single file is structured, and what to replace when building the real platform.

---

## 1. What this is (and isn't)

This is the **couple-facing self-serve venue page** — Mode 1 in the product spec, the most important screen. It is a **single, self-contained HTML file**: no build step, no framework, no dependencies to install. It runs by opening it in any modern browser or hosting it as a static file.

It is a *prototype*, not production code. The architecture is deliberately simple (vanilla JS, data-driven rendering) so it reads in one sitting and can be cannibalized for parts. It is **not** the recommended production stack — see §6.

External runtime dependencies (all over HTTPS, all CDN, no keys):

- **Google Fonts** — Cormorant Garamond (display), Baskervville (editorial body), Inter (UI).
- **SuperSplat** — the live gaussian splat on the Overview (`https://superspl.at/s?id=d95730eb`, the existing Doheny aerial used as placeholder).
- **Unsplash** — placeholder photography for galleries, per-space 3D placeholders, and the director headshot. Every `<img>` has an `onerror` fallback to a styled gradient, so a failed image degrades gracefully rather than breaking layout.

---

## 2. Locked design decisions

These four were agreed before building and govern every screen that follows:

1. **App shell, splat-forward.** One persistent left rail is the *only* chrome. The splat/media fills everything to its right. Nothing floats over the media except a minimal crumb + glance chips and (in 3D mode) the staging control. This is the deliberate answer to "keep it from getting busy."
2. **Persistent space sidebar.** Navigation lives in the rail: Overview + the five spaces, each expanding to **Explore 3D / Gallery / Details**. The rail drives what the main stage shows, so the splat, photos, and copy never compete on screen at once.
3. **Cleaner light app, warmed.** A brighter, higher-contrast app feel (not the editorial-embed look) but pulled away from SaaS: warm neutrals, a single muted-bronze accent (no crimson), serif reserved for display moments only.
4. **Co-branded, venue-forward.** The venue wordmark leads the rail; "Powered by Interlink" sits quietly beneath. Couples trust the venue's name; Interlink builds credibility underneath.

---

## 3. Design system / tokens

All tokens live in `:root` at the top of the `<style>` block.

| Token | Value | Use |
|---|---|---|
| `--bg` | `#F4F1EC` | App background (warm ivory) |
| `--stage-bg` | `#EDE9E2` | Splat stage backdrop |
| `--rail` | `#FFFFFF` | Left rail surface |
| `--ink` | `#2A2724` | Primary text |
| `--ink-soft` | `#6E665C` | Secondary text |
| `--ink-faint` | `#9A9189` | Tertiary / labels |
| `--gold` / `--gold-deep` | `#9A7B4F` / `#7E6240` | Accent (active states, hovers) |
| `--cta` | `#2A2724` | Primary buttons (charcoal) |
| `--line` | `rgba(42,39,36,0.09)` | Hairline borders |

**Type roles:** Cormorant Garamond = display (venue name, space titles, section heads). Baskervville = editorial body (descriptions, feature lists). Inter = all UI (nav, labels, buttons, chips).

**Glass:** floating overlays (crumb, chips, staging, hint) use `backdrop-filter: blur(14px)` over a `rgba(255,255,255,0.72)` fill with a thin white border — the liquid-glass language, dialed back for usability.

---

## 4. File structure (within the single HTML)

```
<head>
  :root tokens + all component CSS (~600 lines, organized by component)
<body>
  .rail        — brand / nav (JS-rendered) / contact card + CTA
  .stage       — crumb + chips, then three stacked <section class="view">:
                   #view-3d       (splat host + staging pills)
                   #view-gallery  (masonry photo grid)
                   #view-details  (editorial copy + specs + features)
  #overlay     — scheduling modal (date → time → form → confirmation)
  <script>     — data + render functions
```

### Rendering model

The whole page is **data-driven** from one array, `SPACES`. Each entry is the schema below; nav and all three stage views are generated from it. To add/edit a space, edit the data — no markup changes.

```js
{
  id:       "garden",
  name:     "The Formal Garden",
  meta:     "Outdoor · 250 seated",      // rail subtitle
  tag:      "Ceremony",                    // crumb tag
  live:     false,                         // true = real SuperSplat iframe; false = photo placeholder
  photo:    "photo-XXXX?w=1400&q=80",      // Unsplash id for the 3D placeholder (when live:false)
  chips:    [["cap","250 seated"],["loc","Outdoor"]],   // glance chips (icon key + label)
  kicker:   "Ceremony Space",              // details kicker
  title:    "The Formal Garden",
  body:     "...",                         // editorial description
  specs:    [["250","Seated","Ceremony"], ...],         // up to 3 stat cells
  features: ["...", "..."],                // bulleted feature list
  gallery:  [["photo-XXXX?w=600&q=80","Caption"], ...], // Unsplash id + caption
  staging:  ["Empty","Ceremony","Reception"]            // pill options, or null
}
```

### Key functions

- `renderNav()` / `renderStage()` — rebuild rail and stage from `current` + `mode`.
- `selectView(id, mode)` — the single entry point for navigation (mode = `3d` | `gallery` | `details`).
- `render3D()` — injects the live splat iframe (`live:true`) or a photo placeholder + staging pills (`live:false`).
- `openModal()` / `confirmBooking()` — scheduling mock with a date strip, time slots, minimal form, and a confirmation state.

---

## 5. What's mocked (replace for production)

| Element | Now | Production |
|---|---|---|
| Per-space 3D | One live splat (Overview); other spaces use a labeled photo placeholder | One gaussian splat capture per space, self-hosted (spec §9.3: Cloudflare R2 + self-hosted SuperSplat viewer, MIT-licensed) |
| Staging toggles | Visual pills, no effect | Swap splat capture / photo set per configuration |
| Scheduling | Static date/time picker + fake confirm | Cal.com / Calendly / Acuity embed or API (spec §5.1.3) |
| Photography | Unsplash placeholders | Venue's curated professional + real-event photos |
| Director headshot | Unsplash placeholder | Real headshot |
| Content | The Doheny Estate (real spaces, descriptions & photos ported from the production tour) | CMS-driven per-venue content (spec §5.3.1) |
| Per-space splats | Overview, Formal Garden & Inner Courtyard are live SuperSplat embeds (ported from the existing tour); Cypress Lane & Front Terrace show real estate photos labelled "3D capture · coming soon" (they were placeholders in the production tour) | One captured splat per space |
| Beverly Hills crest | Stylized SVG stand-in (the official trademarked mark was not available as a file to embed) | Replace with the official asset, base64-embedded |

---

## 6. Notes toward the real platform

This file intentionally stops at Phase 1 (self-serve page). For the build proper, the spec (§9) points to a standalone web app rather than a Squarespace injection. The data-driven `SPACES` model here maps cleanly onto a CMS/API: the schema in §4 is essentially the content model for a space. The view-switching pattern (rail drives a single stage) is the IA worth preserving regardless of framework. The hard parts that this prototype does *not* attempt — real-time state sync for the guided tour (WebSocket/WebRTC) and embedded video (Daily/LiveKit/Twilio) — are Phase 2 and live in the spec, not here.

Remaining brief screens not yet built: Scheduling flow (full), Guided Tour view, Dashboard (analytics), Dashboard (media management), plus mobile-responsive versions of the couple-facing screens.

---

## 7. Running & hosting

**Locally:** open the `.html` file in a browser. (The live splat needs a network connection and WebGPU/WebGL — it won't render in headless screenshot tools but works in normal browsers.)

**Hosting:** it's a single static file, so any static host works with zero config — Netlify Drop, GitHub Pages, Vercel, or Cloudflare Pages. See the chat message accompanying this handoff for step-by-step options.
