# PocketChomp Homepage Redesign Design

Date: 2026-04-20

## Goal

Replace the inherited agency-style homepage with a PocketChomp-first marketing homepage that matches the structure and tone of `design/Main Design - Dark.png` and `design/Main Design - Light.png`, while selectively borrowing implementation-ready patterns from `design/Landing Page v2.html`.

The redesign must preserve the repo's Astro-first, Markdown-driven architecture and prepare the site for future expansion into a larger app marketing site.

## Source Of Truth

- Primary design reference: `design/Main Design - Dark.png`
- Secondary visual reference: `design/Main Design - Light.png`
- Supplemental implementation reference: `design/Landing Page v2.html`

The two PNG mockups define the section structure, pacing, and overall art direction. The HTML reference contributes reusable interaction or section ideas only when they support the mockup and fit the content architecture.

## Scope

This redesign pass covers the homepage only.

Included:

- Homepage structure and section set
- Homepage copy model and content schemas
- Theme system needed to support light and dark homepage variants
- Shared primitives required by the homepage and likely future marketing pages
- Cleanup or replacement of agency-era homepage assumptions

Excluded for this pass:

- Full cleanup of every agency-derived file across the repo
- Redesign of non-homepage routes
- New app functionality beyond homepage marketing and theme preference

## Product And UX Direction

The homepage should feel product-led, local-first, and opinionated.

It should communicate:

- PocketChomp is a nutrition tracker for Canadians
- The product is privacy-conscious and user-first
- The visual identity is bold, clean, and app-centric rather than agency-centric
- The page should support either light or dark presentation without changing structure

The dark version is the stronger brand expression and should drive tone. The light version is mainly a readability and contrast reference. The shipped site should support both via a shared token system.

## Architecture

### Page Composition

`src/pages/index.astro` remains intentionally thin. It should only compose homepage section components and avoid containing large content or styling logic.

Proposed homepage sections:

1. Header with logo, theme toggle, and primary CTA
2. Hero with headline, supporting copy, CTA, and product visual collage
3. Social proof / testimonial strip
4. Flagship feature block for the main modular product story
5. Supporting value grid for privacy, ads, local-first, and Canadian positioning
6. "Beyond the calorie" spotlight section
7. Signup / email CTA section
8. Footer

### Section Ownership

Each section component should continue to load its own content using `getEntry()` or `getCollection()`. This preserves the existing render flow and keeps section ownership clear.

### Shared Primitives

Shared UI primitives should be extracted or preserved only where they support reuse across future pages:

- Button
- Theme toggle
- Section shell/container
- Eyebrow/pill badge
- Testimonial card
- Value or feature card
- Signup form shell

Homepage-specific visuals that are unlikely to be reused can stay section-local.

## Theme System

### Behavior

- Default theme behavior: follow `prefers-color-scheme`
- The site should apply the correct theme before paint to avoid flash
- Users should be able to explicitly switch themes with a homepage toggle
- An explicit user choice should persist across visits
- Light and dark modes must share the exact same structure and content

### Implementation Boundary

Theme logic belongs in the base layout and global styling layer, not inside individual homepage sections.

The implementation should:

- Set a theme attribute or class on the root document early
- Use global design tokens to define light and dark values
- Keep section and component markup shared across both themes
- Avoid duplicating section CSS or Astro markup for each mode

## Content Model

### Principle

Editable marketing copy must remain in Markdown collections with matching Zod schemas in `src/content.config.ts`.

### Homepage Content Changes

The current landing-page collections should be reviewed and either repurposed or replaced to match the new PocketChomp narrative. Agency-era homepage concepts such as portfolio, comparison, and pricing should not survive unchanged if they no longer represent the product story.

Expected content collections for the redesigned homepage:

- Hero singleton
- Testimonials collection or singleton-backed testimonial list
- Flagship feature singleton
- Value pillars collection or singleton-backed repeated cards
- Beyond-the-calorie singleton
- Signup CTA singleton

The exact collection names can be finalized during planning, but the content shape must map cleanly to the new section structure.

### Markdown vs Component Responsibility

- Markdown stores copy, labels, badges, CTA text, repeated item data, and stat/value content
- Astro components handle layout, decorative structure, local imagery, and reusable presentation primitives
- Large editable copy blocks must not be hardcoded into component source

## Visual System

### Tokens And Styling

The redesign must stay aligned with `src/styles/global.css` and the existing OKLCH token approach. The implementation should evolve the token layer as needed rather than replacing it with ad hoc color values.

Typography direction remains:

- Headings: Montserrat Black
- Body copy: Geist

### Design Intent To Preserve

- Strong hero contrast and large uppercase headline treatment
- Rounded, app-like visual forms
- Clear sectional pacing with alternating emphasis
- Product-forward layout rather than service-business layout
- Selective use of orange for high-intent CTAs and feature emphasis
- Green as the core product identity color

## Cleanup Strategy

The homepage should be rebuilt as PocketChomp-first, not reskinned from the previous agency homepage.

During implementation:

- Reuse only patterns that are structurally sound and still relevant
- Remove or rewrite homepage assumptions that reflect an agency site
- Keep any extracted primitives generic enough for future marketing pages

Full deletion of all agency-era code outside the homepage path is deferred until the homepage is in a stable state.

## Risks

### Architectural Risks

- Preserving too much agency-era homepage structure and ending up with the wrong abstractions
- Hardcoding new copy in Astro files instead of evolving content collections
- Duplicating theme-specific section markup

### UX Risks

- Theme flash on initial load
- Insufficient contrast in one theme after token translation
- Mobile layout breakdown in the hero collage and large feature sections

## Verification

There is no dedicated test runner in this repo, so verification should be explicit and evidence-based.

Required checks for implementation completion:

- `npm run build` succeeds
- Homepage renders correctly in static Astro output
- Theme initializes without noticeable flash or mismatch
- Theme toggle persists explicit user preference
- Mobile and desktop layouts hold together across the homepage
- Homepage editable copy remains Markdown-driven with matching Zod schemas

## Implementation Outcome

A successful redesign results in:

- A homepage that matches the new PocketChomp design direction
- A layout-level theme system that future pages can reuse
- A cleaner PocketChomp-first content model for homepage sections
- Reusable shared primitives that support later site expansion
