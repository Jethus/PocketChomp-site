# PocketChomp Homepage Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the homepage as a PocketChomp-first, theme-aware Astro landing page that matches the approved dark/light designs while keeping all editable marketing copy in Markdown collections.

**Architecture:** Keep `src/pages/index.astro` thin and section-driven, move theme behavior into `BaseLayout.astro` and `src/styles/global.css`, and replace the agency-style landing sections with new PocketChomp-focused section components that each load their own content. Promote only the primitives that should be reusable on later marketing pages, and move the new Android frame images into `src/assets/` so production code does not depend on `design/`.

**Tech Stack:** Astro 6, Astro content collections, Tailwind CSS v4, Markdown content files, Astro asset imports

---

### Task 1: Build The Theme Foundation And Shared Shell Primitives

**Files:**
- Modify: `src/layouts/BaseLayout.astro`
- Modify: `src/styles/global.css`
- Modify: `src/components/ui/Button.astro`
- Create: `src/components/ui/ThemeToggle.astro`
- Create: `src/components/ui/SectionShell.astro`
- Create: `src/components/ui/EyebrowPill.astro`

- [ ] **Step 1: Replace the bare layout body with root theme wiring and a no-flash inline script**

```astro
---
import "../styles/global.css";
import { SITE_TITLE, SITE_DESCRIPTION } from "../consts";
import type { ImageMetadata } from "astro";
import { Font } from "astro:assets";

interface Props {
  title: string;
  description?: string;
  image?: ImageMetadata;
}

const canonicalURL = new URL(Astro.url.pathname, Astro.site);
const { title, description = SITE_DESCRIPTION, image } = Astro.props;
const pageTitle = title === SITE_TITLE ? title : `${title} | ${SITE_TITLE}`;

const themeInitScript = `
(() => {
  const storageKey = "pocketchomp-theme";
  const root = document.documentElement;
  const saved = localStorage.getItem(storageKey);
  const systemDark = window.matchMedia("(prefers-color-scheme: dark)").matches;
  const theme = saved === "light" || saved === "dark"
    ? saved
    : systemDark
      ? "dark"
      : "light";
  root.dataset.theme = theme;
  root.style.colorScheme = theme;
})();
`;
---

<!doctype html>
<html lang="en" class="scroll-smooth">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <script is:inline set:html={themeInitScript} />
  </head>
  <body class="min-h-screen bg-app text-app-ink font-body antialiased transition-colors duration-300">
    <slot />
  </body>
</html>
```

- [ ] **Step 2: Replace the old brutalist token set with light/dark semantic tokens in `src/styles/global.css`**

```css
@theme {
  --font-heading: var(--font-montserrat), "Arial Black", "Arial", sans-serif;
  --font-body: var(--font-geist), "Segoe UI", "Helvetica Neue", Arial, sans-serif;

  --color-app-bg: oklch(0.98 0.01 106);
  --color-app-bg-elevated: oklch(0.95 0.01 106);
  --color-app-panel: oklch(0.99 0.005 100);
  --color-app-panel-strong: oklch(0.42 0.08 157);
  --color-app-ink: oklch(0.19 0.015 145);
  --color-app-ink-muted: oklch(0.42 0.02 145);
  --color-app-line: oklch(0.88 0.01 106);
  --color-brand-green: oklch(0.52 0.09 157);
  --color-brand-green-strong: oklch(0.44 0.08 157);
  --color-brand-orange: oklch(0.7 0.2 40);
  --color-brand-orange-ink: oklch(0.2 0.03 40);
  --color-chip-bg: oklch(0.99 0.003 100 / 0.9);
  --radius-card: 1.75rem;
  --radius-pill: 9999px;
  --shadow-soft: 0 12px 30px color-mix(in oklab, black 12%, transparent);
}

@layer base {
  :root[data-theme="dark"] {
    --color-app-bg: oklch(0.14 0.01 145);
    --color-app-bg-elevated: oklch(0.17 0.012 145);
    --color-app-panel: oklch(0.19 0.01 145);
    --color-app-panel-strong: oklch(0.42 0.08 157);
    --color-app-ink: oklch(0.95 0.01 106);
    --color-app-ink-muted: oklch(0.74 0.01 106);
    --color-app-line: oklch(0.26 0.01 145);
    --color-chip-bg: oklch(0.2 0.01 145 / 0.92);
  }

  body {
    background: var(--color-app-bg);
    color: var(--color-app-ink);
    font-family: var(--font-body);
  }
}

@utility bg-app { background-color: var(--color-app-bg); }
@utility bg-app-elevated { background-color: var(--color-app-bg-elevated); }
@utility bg-app-panel { background-color: var(--color-app-panel); }
@utility bg-app-panel-strong { background-color: var(--color-app-panel-strong); }
@utility text-app { color: var(--color-app-ink); }
@utility text-app-ink { color: var(--color-app-ink); }
@utility text-app-muted { color: var(--color-app-ink-muted); }
@utility border-app { border-color: var(--color-app-line); }
@utility text-brand-orange { color: var(--color-brand-orange); }
@utility bg-brand-orange { background-color: var(--color-brand-orange); }
@utility bg-chip { background-color: var(--color-chip-bg); }
```

- [ ] **Step 3: Update `Button.astro` to a reusable variant API that matches the new homepage**

```astro
---
interface Props {
  href?: string;
  variant?: "primary" | "secondary" | "ghost";
  size?: "sm" | "md";
  class?: string;
}

const {
  href,
  variant = "primary",
  size = "md",
  class: className = "",
} = Astro.props;

const base =
  "inline-flex items-center justify-center rounded-full font-heading text-sm uppercase tracking-tight transition-transform duration-150 hover:-translate-y-0.5";
const variants = {
  primary: "bg-brand-orange text-[var(--color-brand-orange-ink)] shadow-[var(--shadow-soft)]",
  secondary: "bg-app-panel text-app-ink border border-app",
  ghost: "bg-transparent text-app-ink border border-app",
};
const sizes = {
  sm: "px-5 py-3",
  md: "px-7 py-4",
};
const classes = `${base} ${variants[variant]} ${sizes[size]} ${className}`;
---

{
  href ? (
    <a href={href} class={classes}><slot /></a>
  ) : (
    <button type="button" class={classes}><slot /></button>
  )
}
```

- [ ] **Step 4: Add the theme toggle primitive**

```astro
---
const label = "Toggle color theme";
---

<button
  type="button"
  class="inline-flex h-12 w-12 items-center justify-center rounded-full border border-app bg-chip text-app-ink"
  aria-label={label}
  data-theme-toggle
>
  <span class="sr-only">{label}</span>
  <svg data-theme-icon="sun" viewBox="0 0 24 24" class="h-5 w-5"><circle cx="12" cy="12" r="4" fill="currentColor" /></svg>
  <svg data-theme-icon="moon" viewBox="0 0 24 24" class="hidden h-5 w-5"><path d="M21 12.8A9 9 0 1 1 11.2 3 7 7 0 0 0 21 12.8Z" fill="currentColor" /></svg>
</button>

<script is:inline>
  const storageKey = "pocketchomp-theme";
  const root = document.documentElement;
  const button = document.querySelector("[data-theme-toggle]");
  const sun = button?.querySelector('[data-theme-icon="sun"]');
  const moon = button?.querySelector('[data-theme-icon="moon"]');

  const syncIcons = () => {
    const dark = root.dataset.theme === "dark";
    sun?.classList.toggle("hidden", dark);
    moon?.classList.toggle("hidden", !dark);
  };

  syncIcons();

  button?.addEventListener("click", () => {
    const next = root.dataset.theme === "dark" ? "light" : "dark";
    root.dataset.theme = next;
    root.style.colorScheme = next;
    localStorage.setItem(storageKey, next);
    syncIcons();
  });
<\/script>
```

- [ ] **Step 5: Add the shared section shell and eyebrow pill primitives**

```astro
--- src/components/ui/SectionShell.astro
interface Props {
  class?: string;
  innerClass?: string;
}

const { class: className = "", innerClass = "" } = Astro.props;
---

<section class={`px-5 py-16 md:px-8 md:py-24 ${className}`}>
  <div class={`mx-auto w-full max-w-7xl ${innerClass}`}>
    <slot />
  </div>
</section>
```

```astro
--- src/components/ui/EyebrowPill.astro
interface Props {
  label: string;
  class?: string;
}

const { label, class: className = "" } = Astro.props;
---

<span class={`inline-flex rounded-full bg-[var(--color-brand-green)] px-4 py-2 text-xs font-heading uppercase tracking-wide text-white ${className}`}>
  {label}
</span>
```

- [ ] **Step 6: Verify the foundation changes compile before homepage work starts**

Run: `npm run build`

Expected: Build completes successfully, and existing pages still render with the new token names and layout script in place.


### Task 2: Replace The Homepage Content Model And Promote The Android Frames Into Production Assets

**Files:**
- Modify: `src/content.config.ts`
- Create: `src/content/landing/social-proof.md`
- Create: `src/content/landing/modular-feature.md`
- Create: `src/content/landing/value-pillars.md`
- Create: `src/content/landing/beyond-calorie.md`
- Create: `src/content/landing/signup-cta.md`
- Modify: `src/content/landing/hero.md`
- Delete: `src/content/landing/what-we-offer.md`
- Delete: `src/content/landing/portfolio.md`
- Delete: `src/content/landing/comparison.md`
- Delete: `src/content/landing/pricing.md`
- Delete: `src/content/landing/cta.md`
- Create: `src/assets/png/android-frame-dark.png`
- Create: `src/assets/png/android-frame-light.png`

- [ ] **Step 1: Rewrite `src/content.config.ts` around PocketChomp homepage sections**

```ts
const landingHero = defineCollection({
  loader: glob({ pattern: "hero.md", base: "src/content/landing" }),
  schema: z.object({
    eyebrow: z.string(),
    headline: z.string(),
    accent: z.string(),
    description: z.string(),
    primaryCtaText: z.string(),
    primaryCtaLink: z.string(),
    secondaryCtaText: z.string(),
    secondaryCtaLink: z.string(),
    heroStats: z.array(
      z.object({
        label: z.string(),
        value: z.string(),
      })
    ),
    collageCards: z.array(
      z.object({
        title: z.string(),
        body: z.string().optional(),
        kind: z.enum(["blank", "ring", "database", "bar"]),
      })
    ),
  }),
});

const landingSocialProof = defineCollection({
  loader: glob({ pattern: "social-proof.md", base: "src/content/landing" }),
  schema: z.object({
    eyebrow: z.string(),
    headline: z.string(),
    description: z.string(),
    testimonials: z.array(
      z.object({
        quote: z.string(),
        initials: z.string(),
        name: z.string(),
        role: z.string(),
        accent: z.enum(["green", "orange"]),
      })
    ),
  }),
});

const landingModularFeature = defineCollection({
  loader: glob({ pattern: "modular-feature.md", base: "src/content/landing" }),
  schema: z.object({
    eyebrow: z.string(),
    headline: z.string(),
    description: z.string(),
    bullets: z.array(z.string()),
  }),
});

const landingValuePillars = defineCollection({
  loader: glob({ pattern: "value-pillars.md", base: "src/content/landing" }),
  schema: z.object({
    items: z.array(
      z.object({
        eyebrow: z.string(),
        headline: z.string(),
        description: z.string(),
        chips: z.array(z.string()),
      })
    ),
  }),
});

const landingBeyondCalorie = defineCollection({
  loader: glob({ pattern: "beyond-calorie.md", base: "src/content/landing" }),
  schema: z.object({
    headline: z.string(),
    description: z.string(),
    nutrients: z.array(z.string()),
    statLabel: z.string(),
    statValue: z.string(),
  }),
});

const landingSignupCta = defineCollection({
  loader: glob({ pattern: "signup-cta.md", base: "src/content/landing" }),
  schema: z.object({
    headline: z.string(),
    description: z.string(),
    inputPlaceholder: z.string(),
    buttonText: z.string(),
    note: z.string(),
  }),
});
```

- [ ] **Step 2: Replace the hero Markdown with PocketChomp-first data**

```md
---
eyebrow: Private beta · iOS & Android
headline: Track your macros,
accent: not your data.
description: The local-first nutrition tracker built for Canadians. Free barcode scanning, a hyper-accurate Canadian database, zero gamification and zero ads.
primaryCtaText: Join the beta
primaryCtaLink: "#signup"
secondaryCtaText: See how it works
secondaryCtaLink: "#modular"
heroStats:
  - label: Local-first
    value: On-device by default
  - label: Built here
    value: Toronto, Canada
collageCards:
  - title: Macro snapshot
    kind: blank
  - title: Logging flow
    kind: blank
  - title: High-contrast nutrient logic
    kind: ring
  - title: Canadian database
    body: Locally sourced product data
    kind: database
---
```

- [ ] **Step 3: Add the remaining landing Markdown files**

```md
--- src/content/landing/social-proof.md
eyebrow: Early insights
headline: Why they’re waiting
description: Real feedback from early supporters.
testimonials:
  - quote: I’ve been looking for a tracker that doesn’t treat my food log like ad inventory. This is it.
    initials: ML
    name: Melissa L.
    role: Beta tester | Toronto
    accent: green
  - quote: Finally, a proper Canadian nutrition database with no guessing if it’s a US version.
    initials: SC
    name: Sarah C.
    role: Beta tester | Belleville
    accent: orange
  - quote: The privacy-first angle is what made me sign up. The clean design kept me reading.
    initials: DK
    name: David K.
    role: Beta tester | Ottawa
    accent: green
---
```

```md
--- src/content/landing/modular-feature.md
eyebrow: Founding members get early access
headline: Fully modular
description: Make the app work for you. Set goals around the nutrition you actually track and rearrange the experience around what matters most.
bullets:
  - Reorder widgets around your own priorities
  - Track macros, micros, and custom daily focus metrics
  - View progress and history without clutter
  - Keep your nutrition flow adaptable as goals change
---
```

```md
--- src/content/landing/value-pillars.md
items:
  - eyebrow: Transparency
    headline: Privacy is our foundation.
    description: We will never sell your data to advertisers, ad brokers, or anyone else.
    chips:
      - No sign-up required
      - 100% on-device
  - eyebrow: Built for the people
    headline: No ads, ever.
    description: The free version should still feel premium. Nothing intrusive, nothing manipulative.
    chips:
      - 0 ads
      - Premium feel
  - eyebrow: Local first
    headline: Choose where your data lives.
    description: Use the app locally only, or opt in to cloud sync later on your terms.
    chips:
      - Your data
      - Your choice
  - eyebrow: Made in Toronto
    headline: Proudly Canadian.
    description: Crafted in Toronto with a database grounded in Canadian products and local grocery reality.
    chips:
      - Canadian products
      - Local groceries
---
```

```md
--- src/content/landing/beyond-calorie.md
headline: Beyond the calorie.
description: Your body is unique. Track what matters to you whether that’s caffeine, fiber, electrolytes, or specific micronutrients.
nutrients:
  - Protein
  - Fiber
  - Caffeine
  - Electrolytes
statLabel: Daily focus
statValue: 180MG
---
```

```md
--- src/content/landing/signup-cta.md
headline: Stay in the loop
description: Launch updates only. No spam. Unsubscribe anytime.
inputPlaceholder: you@email.com
buttonText: Subscribe
note: We only send PocketChomp launch and beta updates.
---
```

- [ ] **Step 4: Copy the Android frame references from `design/assets/` into production assets**

Run:

```powershell
Copy-Item -LiteralPath "C:\repos\PocketChomp-site\design\assets\Android Compact - Black.png" -Destination "C:\repos\PocketChomp-site\src\assets\png\android-frame-dark.png"
Copy-Item -LiteralPath "C:\repos\PocketChomp-site\design\assets\Android Compact - Silver.png" -Destination "C:\repos\PocketChomp-site\src\assets\png\android-frame-light.png"
```

Expected: `src/assets/png/android-frame-dark.png` and `src/assets/png/android-frame-light.png` exist and can be imported by Astro components.

- [ ] **Step 5: Remove the obsolete homepage content files once replacement files exist**

Run:

```powershell
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\content\landing\what-we-offer.md"
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\content\landing\portfolio.md"
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\content\landing\comparison.md"
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\content\landing\pricing.md"
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\content\landing\cta.md"
```

Expected: The old agency-era homepage content entries are gone, and no non-homepage collections are touched.

- [ ] **Step 6: Run a schema-only build check**

Run: `npm run build`

Expected: The build fails only because section components are still querying the old collections, not because the new content schema is invalid.


### Task 3: Replace The Header, Hero, And Social Proof Sections

**Files:**
- Modify: `src/components/Header.astro`
- Modify: `src/components/Hero.astro`
- Create: `src/components/ui/TestimonialCard.astro`
- Create: `src/components/SocialProof.astro`
- Modify: `src/pages/index.astro`

- [ ] **Step 1: Rewrite the header as a reusable PocketChomp site header with CTA and theme toggle**

```astro
---
import Button from "./ui/Button.astro";
import ThemeToggle from "./ui/ThemeToggle.astro";
import logoWhite from "../assets/svgs/logo-white.svg?raw";
import logoBlack from "../assets/svgs/logo-black.svg?raw";
---

<header class="sticky top-0 z-50 border-b border-app bg-[color:color-mix(in_oklab,var(--color-app-bg)_84%,transparent)] backdrop-blur">
  <div class="mx-auto flex max-w-7xl items-center justify-between px-5 py-4 md:px-8">
    <a href="/" class="inline-flex items-center" aria-label="PocketChomp home">
      <span class="dark:hidden" set:html={logoBlack} />
      <span class="hidden dark:inline" set:html={logoWhite} />
    </a>
    <div class="flex items-center gap-3">
      <ThemeToggle />
      <Button href="#signup" size="sm">Join the beta</Button>
    </div>
  </div>
</header>
```

- [ ] **Step 2: Replace `Hero.astro` with the new data-driven hero and theme-aware Android frame**

```astro
---
import { getEntry } from "astro:content";
import SectionShell from "./ui/SectionShell.astro";
import Button from "./ui/Button.astro";
import heroAppShot from "../assets/png/hero.png";
import androidFrameDark from "../assets/png/android-frame-dark.png";
import androidFrameLight from "../assets/png/android-frame-light.png";

const entry = await getEntry("landingHero", "hero");
if (!entry) throw new Error("Missing landing hero content");
const { data } = entry;
---

<SectionShell class="pt-8 md:pt-12" innerClass="grid gap-10 lg:grid-cols-[1.05fr_1fr] lg:items-center">
  <div>
    <p class="mb-6 text-sm uppercase tracking-[0.24em] text-app-muted">{data.eyebrow}</p>
    <h1 class="max-w-[10ch] text-6xl uppercase leading-[0.9] md:text-8xl">
      {data.headline}
      <span class="block text-brand-orange">{data.accent}</span>
    </h1>
    <p class="mt-6 max-w-2xl text-lg leading-relaxed text-app-muted">{data.description}</p>
    <div class="mt-8 flex flex-wrap gap-4">
      <Button href={data.primaryCtaLink}>{data.primaryCtaText}</Button>
      <Button href={data.secondaryCtaLink} variant="ghost">{data.secondaryCtaText}</Button>
    </div>
  </div>

  <div class="relative mx-auto w-full max-w-[34rem]">
    <img src={androidFrameLight.src} alt="" class="block dark:hidden" />
    <img src={androidFrameDark.src} alt="" class="hidden dark:block" />
    <img src={heroAppShot.src} alt="PocketChomp app preview" class="absolute inset-[3.2%_5.5%_3.1%_5.5%] h-[93.7%] w-[89%] rounded-[2.4rem] object-cover" />
  </div>
</SectionShell>
```

- [ ] **Step 3: Add the testimonial card primitive and the social proof strip section**

```astro
--- src/components/ui/TestimonialCard.astro
interface Props {
  quote: string;
  initials: string;
  name: string;
  role: string;
  accent: "green" | "orange";
}

const { quote, initials, name, role, accent } = Astro.props;
const accentClass = accent === "orange" ? "bg-brand-orange text-[var(--color-brand-orange-ink)]" : "bg-[var(--color-brand-green)] text-white";
---

<article class="min-w-[20rem] rounded-[1.75rem] border border-app bg-app-panel p-6 shadow-[var(--shadow-soft)] md:min-w-[24rem]">
  <p class="text-lg leading-relaxed text-app-ink">{quote}</p>
  <div class="mt-6 flex items-center gap-4">
    <span class={`inline-flex h-14 w-14 items-center justify-center rounded-full font-mono text-lg ${accentClass}`}>{initials}</span>
    <div>
      <p class="font-heading text-xl">{name}</p>
      <p class="text-app-muted">{role}</p>
    </div>
  </div>
</article>
```

```astro
--- src/components/SocialProof.astro
import { getEntry } from "astro:content";
import SectionShell from "./ui/SectionShell.astro";
import EyebrowPill from "./ui/EyebrowPill.astro";
import TestimonialCard from "./ui/TestimonialCard.astro";

const entry = await getEntry("landingSocialProof", "social-proof");
if (!entry) throw new Error("Missing social proof content");
const { data } = entry;
---

<SectionShell innerClass="space-y-10">
  <div class="space-y-4">
    <EyebrowPill label={data.eyebrow} />
    <h2 class="text-4xl uppercase md:text-6xl">{data.headline}</h2>
    <p class="max-w-2xl text-lg text-app-muted">{data.description}</p>
  </div>
  <div class="flex gap-5 overflow-x-auto pb-2">
    {data.testimonials.map((item) => <TestimonialCard {...item} />)}
  </div>
</SectionShell>
```

- [ ] **Step 4: Update `src/pages/index.astro` to use the new top-of-page section set**

```astro
---
import BaseLayout from "../layouts/BaseLayout.astro";
import Header from "../components/Header.astro";
import Hero from "../components/Hero.astro";
import SocialProof from "../components/SocialProof.astro";
import Footer from "../components/Footer.astro";
import { SITE_TITLE, SITE_DESCRIPTION } from "../consts";
---

<BaseLayout title={SITE_TITLE} description={SITE_DESCRIPTION}>
  <Header />
  <main>
    <Hero />
    <SocialProof />
  </main>
  <Footer />
</BaseLayout>
```

- [ ] **Step 5: Run the homepage build checkpoint**

Run: `npm run build`

Expected: Build still fails because the lower homepage sections still point at deleted agency collections, but the header, hero, and social proof code compiles cleanly.


### Task 4: Build The Modular Feature, Value Pillars, Beyond-Calorie, And Signup Sections

**Files:**
- Create: `src/components/ui/ChipList.astro`
- Create: `src/components/ui/SignupForm.astro`
- Create: `src/components/ModularFeature.astro`
- Create: `src/components/ValuePillars.astro`
- Create: `src/components/BeyondCalorie.astro`
- Modify: `src/components/ClosingCTA.astro`
- Modify: `src/components/Footer.astro`
- Modify: `src/pages/index.astro`

- [ ] **Step 1: Add the chip list and signup form UI primitives**

```astro
--- src/components/ui/ChipList.astro
interface Props {
  items: string[];
  class?: string;
}

const { items, class: className = "" } = Astro.props;
---

<ul class={`flex flex-wrap gap-3 ${className}`}>
  {items.map((item) => (
    <li class="rounded-full border border-app bg-chip px-4 py-3 text-sm font-semibold text-app-ink">
      {item}
    </li>
  ))}
</ul>
```

```astro
--- src/components/ui/SignupForm.astro
interface Props {
  placeholder: string;
  buttonText: string;
}

const { placeholder, buttonText } = Astro.props;
---

<form id="signup" class="grid gap-3 md:grid-cols-[1fr_auto]">
  <label class="sr-only" for="signup-email">Email address</label>
  <input
    id="signup-email"
    type="email"
    placeholder={placeholder}
    class="h-14 rounded-full border border-[var(--color-brand-orange)] bg-app-panel px-6 text-app-ink placeholder:text-app-muted"
  />
  <button type="submit" class="h-14 rounded-full bg-brand-orange px-8 font-heading uppercase text-[var(--color-brand-orange-ink)]">
    {buttonText}
  </button>
</form>
```

- [ ] **Step 2: Implement the modular feature and value pillars sections**

```astro
--- src/components/ModularFeature.astro
import { getEntry } from "astro:content";
import SectionShell from "./ui/SectionShell.astro";
import EyebrowPill from "./ui/EyebrowPill.astro";
import phoneShot from "../assets/png/hero.png";
import androidFrameDark from "../assets/png/android-frame-dark.png";
import androidFrameLight from "../assets/png/android-frame-light.png";

const entry = await getEntry("landingModularFeature", "modular-feature");
if (!entry) throw new Error("Missing modular feature content");
const { data } = entry;
---

<SectionShell class="pt-6" innerClass="rounded-[2.5rem] bg-app-panel-strong px-6 py-10 md:px-10 md:py-14 lg:grid lg:grid-cols-[0.85fr_1.15fr] lg:items-center">
  <div class="relative mx-auto w-full max-w-sm">
    <img src={androidFrameLight.src} alt="" class="block dark:hidden" />
    <img src={androidFrameDark.src} alt="" class="hidden dark:block" />
    <img src={phoneShot.src} alt="" class="absolute inset-[3.2%_5.5%_3.1%_5.5%] h-[93.7%] w-[89%] rounded-[2.4rem] object-cover" />
  </div>
  <div class="mt-10 lg:mt-0">
    <EyebrowPill label={data.eyebrow} class="bg-white/15 text-white" />
    <h2 id="modular" class="mt-5 text-5xl uppercase text-white md:text-7xl">{data.headline}</h2>
    <p class="mt-6 max-w-2xl text-lg leading-relaxed text-white/80">{data.description}</p>
    <ul class="mt-8 grid gap-4 md:grid-cols-2">
      {data.bullets.map((bullet) => <li class="text-base text-white">{bullet}</li>)}
    </ul>
  </div>
</SectionShell>
```

```astro
--- src/components/ValuePillars.astro
import { getEntry } from "astro:content";
import SectionShell from "./ui/SectionShell.astro";
import EyebrowPill from "./ui/EyebrowPill.astro";
import ChipList from "./ui/ChipList.astro";

const entry = await getEntry("landingValuePillars", "value-pillars");
if (!entry) throw new Error("Missing value pillars content");
const { data } = entry;
---

<SectionShell innerClass="grid gap-10 md:grid-cols-2">
  {data.items.map((item) => (
    <article class="space-y-5">
      <EyebrowPill label={item.eyebrow} />
      <h2 class="max-w-[10ch] text-4xl uppercase md:text-6xl">{item.headline}</h2>
      <p class="max-w-xl text-lg leading-relaxed text-app-muted">{item.description}</p>
      <ChipList items={item.chips} />
    </article>
  ))}
</SectionShell>
```

- [ ] **Step 3: Build the beyond-calorie section and reuse `ClosingCTA.astro` as the signup CTA section**

```astro
--- src/components/BeyondCalorie.astro
import { getEntry } from "astro:content";
import SectionShell from "./ui/SectionShell.astro";

const entry = await getEntry("landingBeyondCalorie", "beyond-calorie");
if (!entry) throw new Error("Missing beyond calorie content");
const { data } = entry;
---

<SectionShell innerClass="rounded-[2.5rem] bg-brand-orange px-6 py-10 text-[var(--color-brand-orange-ink)] md:px-10 md:py-14 lg:grid lg:grid-cols-[0.95fr_1.05fr] lg:items-center">
  <div>
    <h2 class="max-w-[8ch] text-5xl uppercase md:text-7xl">{data.headline}</h2>
    <p class="mt-6 max-w-2xl text-lg leading-relaxed">{data.description}</p>
    <ul class="mt-8 grid gap-3 md:grid-cols-2">
      {data.nutrients.map((nutrient) => <li class="rounded-3xl border border-black/20 px-5 py-4 font-heading text-xl uppercase">{nutrient}</li>)}
    </ul>
  </div>
  <div class="mt-10 rounded-[2rem] bg-white/90 p-6 shadow-[var(--shadow-soft)] lg:mt-0">
    <p class="text-sm uppercase tracking-[0.24em] text-black/60">{data.statLabel}</p>
    <div class="mt-6 space-y-5">
      <div class="space-y-2">
        <div class="h-5 rounded-full bg-black/10"><div class="h-full w-3/4 rounded-full bg-brand-orange"></div></div>
      </div>
      <div class="h-5 rounded-full bg-black/10"><div class="h-full w-2/5 rounded-full bg-[var(--color-brand-green)]"></div></div>
    </div>
    <div class="mt-8 inline-flex items-center rounded-[1.5rem] bg-black px-6 py-4 text-white">
      <span class="text-sm uppercase tracking-[0.2em] text-white/60">Caffeine</span>
      <strong class="ml-4 text-4xl font-heading">{data.statValue}</strong>
    </div>
  </div>
</SectionShell>
```

```astro
--- src/components/ClosingCTA.astro
import { getEntry } from "astro:content";
import SectionShell from "./ui/SectionShell.astro";
import SignupForm from "./ui/SignupForm.astro";

const entry = await getEntry("landingSignupCta", "signup-cta");
if (!entry) throw new Error("Missing signup CTA content");
const { data } = entry;
---

<SectionShell class="pb-10" innerClass="grid gap-8 lg:grid-cols-[1fr_1fr] lg:items-center">
  <div>
    <h2 class="text-4xl uppercase md:text-6xl">{data.headline}</h2>
    <p class="mt-4 max-w-xl text-lg text-app-muted">{data.description}</p>
  </div>
  <div>
    <SignupForm placeholder={data.inputPlaceholder} buttonText={data.buttonText} />
    <p class="mt-4 text-sm text-app-muted">{data.note}</p>
  </div>
</SectionShell>
```

- [ ] **Step 4: Rewrite `Footer.astro` to match the PocketChomp design and keep it shareable**

```astro
---
import logoWhite from "../assets/svgs/logo-white.svg?raw";
import logoBlack from "../assets/svgs/logo-black.svg?raw";
---

<footer class="border-t border-app bg-app-elevated">
  <div class="mx-auto flex max-w-7xl flex-col gap-8 px-5 py-10 md:flex-row md:items-end md:justify-between md:px-8">
    <div class="space-y-4">
      <a href="/" class="inline-flex items-center" aria-label="PocketChomp home">
        <span class="dark:hidden" set:html={logoBlack} />
        <span class="hidden dark:inline" set:html={logoWhite} />
      </a>
      <p class="text-sm text-app-muted">© 2026 PocketChomp. Canadian-made. User-first.</p>
    </div>
    <nav aria-label="Footer" class="flex flex-wrap gap-6 text-sm text-app-muted">
      <a href="/privacy">Privacy Policy</a>
      <a href="/terms">Terms of Service</a>
      <a href="/contact">Contact Us</a>
    </nav>
  </div>
</footer>
```

- [ ] **Step 5: Compose the full homepage**

```astro
---
import BaseLayout from "../layouts/BaseLayout.astro";
import Header from "../components/Header.astro";
import Hero from "../components/Hero.astro";
import SocialProof from "../components/SocialProof.astro";
import ModularFeature from "../components/ModularFeature.astro";
import ValuePillars from "../components/ValuePillars.astro";
import BeyondCalorie from "../components/BeyondCalorie.astro";
import ClosingCTA from "../components/ClosingCTA.astro";
import Footer from "../components/Footer.astro";
import { SITE_TITLE, SITE_DESCRIPTION } from "../consts";
---

<BaseLayout title={SITE_TITLE} description={SITE_DESCRIPTION}>
  <Header />
  <main>
    <Hero />
    <SocialProof />
    <ModularFeature />
    <ValuePillars />
    <BeyondCalorie />
    <ClosingCTA />
  </main>
  <Footer />
</BaseLayout>
```

- [ ] **Step 6: Run the full homepage build**

Run: `npm run build`

Expected: Build passes with the new homepage section set and updated content collections.


### Task 5: Clean Out Unused Homepage Pieces And Verify The Final Experience

**Files:**
- Delete: `src/components/Features.astro`
- Delete: `src/components/Portfolio.astro`
- Delete: `src/components/Comparison.astro`
- Delete: `src/components/Pricing.astro`
- Modify: `src/components/Hero.astro`
- Modify: `src/components/ModularFeature.astro`
- Modify: `src/components/BeyondCalorie.astro`

- [ ] **Step 1: Remove the old homepage-only agency section components after the new homepage is stable**

Run:

```powershell
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\components\Features.astro"
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\components\Portfolio.astro"
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\components\Comparison.astro"
Remove-Item -LiteralPath "C:\repos\PocketChomp-site\src\components\Pricing.astro"
```

Expected: No imported component paths in `src/pages/index.astro` point at deleted files.

- [ ] **Step 2: Tighten responsive layout details for the three most design-sensitive sections**

```astro
--- excerpt for `src/components/Hero.astro`
<SectionShell class="overflow-x-clip pt-8 md:pt-12" innerClass="grid gap-10 lg:grid-cols-[1.05fr_1fr] lg:items-center">
  <!-- keep headline width constrained -->
  <h1 class="max-w-[9ch] text-5xl uppercase leading-[0.9] sm:text-6xl md:text-7xl lg:text-[5.5rem]">
```

```astro
--- excerpt for `src/components/ModularFeature.astro`
<SectionShell class="pt-6" innerClass="rounded-[2.5rem] bg-app-panel-strong px-6 py-10 md:px-10 md:py-14 lg:grid lg:grid-cols-[0.85fr_1.15fr] lg:items-center">
  <div class="relative mx-auto w-full max-w-[18rem] lg:max-w-sm">
```

```astro
--- excerpt for `src/components/BeyondCalorie.astro`
<SectionShell innerClass="rounded-[2.5rem] bg-brand-orange px-6 py-10 text-[var(--color-brand-orange-ink)] md:px-10 md:py-14 lg:grid lg:grid-cols-[0.95fr_1.05fr] lg:items-center">
  <ul class="mt-8 grid gap-3 sm:grid-cols-2">
```

- [ ] **Step 3: Manually verify both themes and the explicit override flow in the browser**

Run:

```powershell
npm run dev
```

Expected:
- First load follows the operating system theme
- Clicking the toggle switches themes immediately
- Refresh preserves the explicit theme choice
- Header, hero, modular section, value pillars, beyond-calorie section, signup CTA, and footer all remain structurally identical between themes

- [ ] **Step 4: Run the production build one final time**

Run: `npm run build`

Expected: PASS with no missing collection errors, no missing asset imports, and a static `dist/` output.

- [ ] **Step 5: Commit the homepage redesign in focused slices**

```bash
git add src/layouts/BaseLayout.astro src/styles/global.css src/components/ui/Button.astro src/components/ui/ThemeToggle.astro src/components/ui/SectionShell.astro src/components/ui/EyebrowPill.astro
git commit -m "feat: add PocketChomp theme foundation"

git add src/content.config.ts src/content/landing/hero.md src/content/landing/social-proof.md src/content/landing/modular-feature.md src/content/landing/value-pillars.md src/content/landing/beyond-calorie.md src/content/landing/signup-cta.md src/assets/png/android-frame-dark.png src/assets/png/android-frame-light.png
git commit -m "feat: add PocketChomp homepage content model"

git add src/components/Header.astro src/components/Hero.astro src/components/SocialProof.astro src/components/ui/TestimonialCard.astro src/components/ModularFeature.astro src/components/ValuePillars.astro src/components/BeyondCalorie.astro src/components/ClosingCTA.astro src/components/ui/ChipList.astro src/components/ui/SignupForm.astro src/components/Footer.astro src/pages/index.astro
git commit -m "feat: rebuild PocketChomp homepage"
```

## Self-Review

### Spec Coverage

- Theme system: covered in Task 1
- Markdown + Zod content model: covered in Task 2
- Homepage section replacement: covered in Tasks 3 and 4
- Android frame assets: covered in Task 2
- Cleanup of homepage-only agency remnants: covered in Task 5
- Build and manual verification: covered in Tasks 1, 2, 4, and 5

### Placeholder Scan

No `TODO`, `TBD`, or deferred implementation markers remain in the plan. Each task names exact files, concrete content structures, and concrete commands.

### Type Consistency

Collection IDs, component names, and asset paths are used consistently across the plan:

- `landingHero`
- `landingSocialProof`
- `landingModularFeature`
- `landingValuePillars`
- `landingBeyondCalorie`
- `landingSignupCta`
- `android-frame-dark.png`
- `android-frame-light.png`
