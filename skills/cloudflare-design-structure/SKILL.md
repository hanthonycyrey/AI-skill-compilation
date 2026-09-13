---
name: cloudflare-design-structure
description: Cloudflare.com's structural design language — grid/layout system, hero composition, bento feature grids, stat-proof bands, 3D/WebGL ambient hero animation, and the signature live agent/workflow status card pattern. Covers STRUCTURE only, not Cloudflare's brand colors or typeface. Use this whenever building or reviewing a landing page, marketing site, or hero section that should feel "Cloudflare-inspired" or "infra-SaaS" — including the Tarbajo marketing site — or whenever the user asks for bento grids, live status/agent cards, real-time-feeling tickers, count-up stat bands, or a network/globe hero animation, even if they don't name Cloudflare directly.
---

# Cloudflare Design Structure

Cloudflare's homepage is built to make abstract infrastructure feel tangible and *alive*. The structural trick isn't any single component — it's that almost every section either shows a number changing, a card streaming in, or motion happening in the background, instead of just describing what the product does. Keep that principle in mind throughout: **show the system working, don't just claim it works.**

This skill covers layout, grid, animation, and real-time UI patterns only. Colors and typography choices are intentionally out of scope — apply whatever brand palette and type the project already uses; only borrow the *structure*.

## Section-by-section anatomy

Use this as the default page skeleton when building a Cloudflare-inspired landing page. Not every project needs all nine sections, but this is the order and rationale:

1. **Minimal top nav** — logo, 3–4 top-level links, one secondary text link (login), one primary button. Sticky, low height, no mega-menu clutter at first glance.

2. **Hero**
   - Full-bleed ambient background: a looping video, canvas/WebGL animation, or (in earlier eras of the brand) a single saturated flat color block. The background communicates "network" without literal explanation.
   - A small rounded "announcement pill" sits above the headline (event, launch, blog post link) — it's how the hero stays fresh without touching the core message.
   - One large, editorial-weight headline. Medium font weight, not heavy/black — it should read like a confident magazine dek, not a shouting SaaS banner. Often phrased as a bold claim or a big number.
   - One short subhead (1–2 sentences) + exactly **one** primary CTA. Resist the urge to add a second competing CTA in the hero.

3. **Proof band directly under the fold** ("Region: Earth" style)
   - A short kicker/eyebrow line.
   - 2–4 stat cards in an *unequal*-width row — some are just a giant number + one-line label, others carry a full descriptive sentence. The unevenness is what makes it feel like real reporting rather than a template.
   - Numbers should look like they're live-measured (e.g. "310B daily threats blocked", "4.5x faster") even when the underlying value is static per page-load.

4. **Social proof band**
   - Auto-scrolling logo marquee (continuous, no visible start/end) paired with a rotating testimonial card that has an explicit pause-on-hover/interaction control. Combining a moving element (marquee) with a static-but-rotating element (quote) is the key pairing — don't make everything move at once.

5. **Bento feature grid** ("Why choose X")
   - Asymmetric CSS grid — a mix of 1×1, 2×1, and 1×2 cells, not a uniform 3-column repeat. The unevenness creates hierarchy: one or two "hero" cards get more visual weight than the rest.
   - Each card: small icon or mono-styled label, one-line heading, short supporting copy. Keep card copy terse — this section is scannable, not read line-by-line.

6. **Live/interactive demonstration section**
   - Take one abstract concept (pricing, latency, throughput) and represent it as a horizontal strip of small units — ticks, cells, or blocks — that animate across the viewport and visibly change state at a threshold (e.g. free → paid once a duration is crossed). This converts a static pricing table into something that feels like a live meter.

7. **Live workflow/agent status cards** — the signature move
   - A kicker line describing overall state in present-continuous tense ("Launching agents to analyze repository…").
   - A running counter of active units ("3 background agents launched") that increments rather than just stating a fixed number.
   - A small stack/row of individual cards, each with: a mono-styled identifier (reads like a process name), a current action in present-continuous form that can update ("scanning PRs", "running checks", "fixed documentation"), and a live-looking metric (token count, request count) that ticks upward.
   - Stagger each card's entrance and updates slightly instead of revealing them all at once — the asynchronous timing is what sells the "real work is happening" illusion.

8. **Closing CTA band** — mirrors the hero: one headline, one CTA, nothing else. Signals "you've reached the end, here's the one action to take."

9. **Mega-footer** — dense, multi-column, organized by audience/purpose category (e.g. Getting started / Company / Compliance / Resources / Developers / Solutions) rather than by product line. It's meant to double as a sitemap.

## Grid & layout system

- A responsive 12-column grid under a generous max-width container (roughly 1280–1440px), with consistent gutters.
- Generous vertical rhythm between full-width sections — large top/bottom padding (roughly 96–160px on desktop, collapsing on mobile) so each section reads as its own "slide" while scrolling.
- Bento/feature grids use explicit `grid-column` / `grid-row` spans per card rather than a uniform flex row — this is what produces the asymmetry described above.
- Stat-card rows vary column count/width by content weight: a card with just "310B / daily threats blocked" is narrower than a neighboring card carrying a full sentence.
- See `references/component-patterns.md` for concrete CSS Grid starting points.

## 3D / ambient hero animation

Cloudflare's brand (across the homepage and product pages like /network) leans heavily on a rotating, interactive network-globe motif — a WebGL globe with glowing points at data-center locations and connecting arcs, commonly built today with three.js or a higher-level wrapper like `react-globe.gl`. The homepage hero itself sometimes uses a simpler looping video/canvas treatment of the same idea rather than a fully interactive globe.

When building a Cloudflare-inspired hero:
- Treat the background as **ambient, not interactive-by-default** — slow, looping motion (particles, connecting lines, a slowly rotating globe) sitting behind the text. It should never compete with headline legibility; dim or blur it if needed.
- Keep foreground UI (headline, subhead, CTA) fully static — no animation on the text itself. The motion budget belongs to the background only.
- Always have a static poster-image fallback for `prefers-reduced-motion` and low-power devices — Cloudflare's own markup ships a `hero-poster` image for exactly this reason.
- If true interactivity is in scope (drag-to-rotate, hover-to-highlight a location), scope it to a dedicated visualization component rather than the homepage hero — that's a heavier, deliberate feature, not incidental hero decoration.

## Real-time / live-status UI pattern (reusable recipe)

This is the pattern behind both the workflow-status cards (section 7) and the pricing ticker (section 6). The recipe generalizes to any product that has a pipeline, a queue, or multiple steps:

1. One kicker sentence stating what's happening right now, present-continuous tense.
2. A counter that increments as units complete or join, rather than a static total.
3. A row/stack of per-unit cards: identifier + current action (which can change over the card's lifetime) + a live-looking metric.
4. Stagger timing across cards/units — asynchronous reveal reads as "real," synchronous reveal reads as "decorative."

See `references/component-patterns.md` for a working React pattern (staggered reveal + incrementing counters) and a count-up stat-card pattern.

## Typography & motion feel (structure only — no specific colors/typefaces prescribed)

- Large, medium-weight display type for headlines — restrained rather than heavy/black. The goal is "confident editorial statement," not "shouting banner."
- A monospace face reserved specifically for: stat captions, status-card identifiers, and any code/data-adjacent label. This is a signal, not decoration — mono means "this is live/technical data," the humanist sans means "this is narrative copy." Keep that split consistent everywhere you apply this skill.
- Micro-interactions: subtle lift/border on card hover, pause-on-hover for carousels, count-up animation triggered on scroll-into-view for stat numbers.

## Applying this to the Tarbajo site

Tarbajo is a career-navigation / resume-parsing platform, so the same structural moves map naturally onto its own pipeline instead of Cloudflare's network:

- **Hero** — ambient background could visualize resume/job matching (a flowing graph of nodes connecting candidates to roles) instead of a network globe; same "confident claim + single CTA" composition.
- **Proof band** — stats like graduates matched, resumes parsed, HEIs/TVIs onboarded, presented as the same uneven stat-card row.
- **Live workflow status cards** — map directly onto the resume-parsing pipeline: cards like "parsing-agent — extracting skills", "matching-agent — ranking roles", "verify-agent — checking HEI credentials", each with a live-feeling counter (resumes processed, matches found). This is the strongest borrow from Cloudflare's pattern because Tarbajo's actual product *is* a multi-step pipeline — the live-status cards can show real (or realistically simulated) pipeline activity rather than being purely decorative.
- **Bento grid** — feature cards for the platform's core value props (semantic parsing, HEI/TVI coverage, employer matching, etc.), asymmetric rather than a uniform 3-up grid.

Keep Tarbajo's own color palette and typography — only the structural moves above are being borrowed from Cloudflare.
