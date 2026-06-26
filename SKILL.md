---
name: web-seo-aeo-geo-google-naver
description: >-
  Implements SEO, AEO, and GEO for Next.js App Router: Google Search Console +
  Naver Search Advisor (sitemap, robots, verification, Naver 80-char metadata),
  plus Answer/Generative Engine optimization (FAQPage, Review, Organization,
  BreadcrumbList JSON-LD, SSR crawlable pages, dynamic OG). Use when deploying
  a web app, setting up search indexing, AI Overviews, LLM citability, 서치어드바이저,
  or when the user asks for SEO/AEO/GEO. Cursor: @web-seo-aeo-geo-google-naver.
  Claude Code: /web-seo-aeo-geo-google-naver. Codex: $web-seo-aeo-geo-google-naver.
---

# Web SEO + AEO + GEO — Google + Naver (Next.js)

Implement production **search indexing (SEO)**, **answer surfaces (AEO)**, and **LLM citability (GEO)** end-to-end: code, env instructions, build verification, and console handoff. Work autonomously after discovery.

## Terminology

| Term | Meaning | Examples |
|------|---------|----------|
| **SEO** | Classic search indexing & ranking | sitemap, meta tags, GSC, Naver |
| **AEO** | Answer Engine Optimization | Google AI Overviews, featured snippets, FAQ rich results |
| **GEO** | Generative Engine Optimization | ChatGPT, Perplexity, Gemini citing your pages |

SEO is the foundation. AEO/GEO layers **structured facts + crawlable HTML** so machines can quote you accurately.

## When to Use

- User deploys a Next.js web app and wants search indexing
- User mentions Google Search Console, Naver Search Advisor (서치어드바이저), sitemap, robots, JSON-LD
- User wants FAQ pages, review schema, AI-friendly entity pages, dynamic OG
- User hit Naver URL inspection warnings (description > 80 chars, og:description mismatch)

**When NOT to use:** non-web projects, audit-only with no code changes requested

---

## Done Criteria

### Google (SEO)
- [ ] `metadata.verification.google` from env
- [ ] `app/sitemap.ts` — all public indexable URLs, https, valid XML
- [ ] `app/robots.ts` — allow `/`, disallow private routes (`/admin`, `/api/`, etc.)
- [ ] Per-page metadata: title, description, canonical, openGraph, twitter
- [ ] JSON-LD on home + detail pages (WebSite, page-appropriate types)
- [ ] SSR/SSG HTML for indexable routes (not SPA-only shells)
- [ ] Private routes: `noindex`

### Naver (SEO)
- [ ] `naver-site-verification` in `metadata.verification.other` (or HTML file for subdomain-only sites)
- [ ] Every indexable page: `description` === `openGraph.description` === `twitter.description`
- [ ] Every indexable page: description ≤ **80 chars** (CJK-aware: count string length)
- [ ] Every indexable page: **unique** description (no site-wide duplicate)
- [ ] Site registered on **https** (not http)
- [ ] Sitemap submittable at Naver: **요청 → 사이트맵 제출**

### AEO + GEO
- [ ] **SSR entity pages** — each indexable place/item has `/entity/[id]` with H1, address, facts, internal links (not `?id=` only)
- [ ] **FAQ route** `/faq` — visible Q&A **matches** `FAQPage` JSON-LD exactly
- [ ] **Organization** JSON-LD on home (brand entity)
- [ ] **BreadcrumbList** JSON-LD where breadcrumbs render
- [ ] **Restaurant/LocalBusiness** JSON-LD on detail pages: name, url, address, geo, telephone, sameAs, aggregateRating when available
- [ ] **Review** JSON-LD — user comments with `reviewBody` rendered in HTML; max ~10 in schema; **no fake reviewRating** unless you store per-review stars
- [ ] **Dynamic OG** — `opengraph-image.tsx` for home/detail; API route for share links with query params (e.g. shared lists)
- [ ] **One canonical URL** per entity (multi-domain: pick primary; aliases canonical-point or 301)
- [ ] Host-aware sitemap when multiple domains serve the same app

### Copy separation (critical)
| Layer | Length | Purpose |
|-------|--------|---------|
| meta / OG / Twitter | ≤ 80 chars | Naver URL inspection |
| JSON-LD / body SEO text | longer OK | Google + AEO/GEO richness |

Reference: https://searchadvisor.naver.com/guide/markup-content

---

## Step 0 — Discovery

1. Read project rules (`AGENTS.md`, etc.) and `app/layout.tsx`
2. Detect stack: Next.js App Router, deployment target (Vercel, etc.), data source
3. Identify indexable routes vs noindex routes
4. Resolve production domain(s) and **canonical primary** per product/locale
5. Check existing verification env vars (do not invent codes):
   - `NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION`
   - `NEXT_PUBLIC_NAVER_SITE_VERIFICATION`
6. Run baseline `npm run build` if feasible
7. Brief plan (SEO → crawlable pages → AEO/GEO schema → OG), then implement

**Ask user only when blocked:** missing domain, missing page types, or verification codes needed for live check

---

## Step 1 — Infrastructure (SEO)

### 1A. `lib/naver-markup.ts`

Short copy for meta + OG only. No import from `lib/seo.ts` (avoid circular deps).

```ts
export const NAVER_DESC_MAX = 80;

export function naverClamp(text: string, max = NAVER_DESC_MAX): string {
  const trimmed = text.trim().replace(/\s+/g, " ");
  if (trimmed.length <= max) return trimmed;
  return `${trimmed.slice(0, max - 1).trimEnd()}…`;
}
```

Export per-page titles/descriptions. Dynamic helpers accept entity data and return clamped strings.

### 1B. `lib/seo.ts`

Long copy + metadata builders + JSON-LD. Import short copy from `naver-markup.ts`.

Every builder must set **identical** `description` on:
- top-level `description`
- `openGraph.description`
- `twitter.description`

### 1C. `app/layout.tsx`

- Home title + description from naver-markup
- Verification block from env (never hardcode real tokens; env-only or empty fallback)
- `metadataBase: new URL(SITE_URL)`

### 1D. `app/robots.ts`

Dynamic robots route. Do **not** duplicate with `public/robots.txt`. Point `Sitemap:` at the **canonical origin** for the current host when multi-domain.

### 1E. `app/sitemap.ts`

All public URLs on the **canonical origin** for that property. https, under Naver limits (≤ 50,000 URLs, ≤ 10 MB).

Adapt route names to the project (`/items/[id]`, `/blog/[slug]`, etc.).

---

## Step 2 — Crawlable pages (SEO + GEO foundation)

If the UI is client-heavy (map, dashboard, canvas app):

- Add SSR/SSG routes with real HTML: headings, text, internal links
- `generateMetadata()` per route using naver short copy
- JsonLd component with long description
- Prefer share URLs like `/items/{id}` over `?id=` query params
- Render **user-generated reviews/comments** in HTML when present in DB

Match existing project conventions for file paths and components.

---

## Step 3 — Per-page checklist (SEO)

For each indexable route:

| Check | Rule |
|-------|------|
| description | ≤ 80 chars |
| og:description | === description |
| twitter.description | === description |
| uniqueness | ≠ home and ≠ other pages |
| canonical | absolute https URL (primary domain) |
| title | reflects page content |

Static pages (privacy, terms): short unique descriptions. Admin: noindex.

---

## Step 4 — AEO + GEO layer

Implement after Step 2 crawlable pages exist.

### 4A. FAQ page (`/faq`)

- Visible `<h1>` + `<dl>` Q&A on page
- `FAQPage` JSON-LD — **same questions and answers** as visible content
- Naver: FAQ meta description ≤ 80 chars, unique
- Optional: per-locale FAQ (ko for Korea apps, en for international)

### 4B. Entity JSON-LD (detail pages)

**Restaurant / LocalBusiness** (adapt type to project):

- `name`, `url`, `description` (long copy OK)
- `address`, `geo`, `telephone`, `sameAs` (only real data)
- `aggregateRating` only when you have rating + count
- `image` from entity photo URLs

**Home:**

- `WebSite` + `SearchAction` if site search exists
- `Organization` — name, url, logo

**Navigation:**

- `BreadcrumbList` matching visible breadcrumb links

### 4C. Review schema (UGC)

```ts
function reviewJsonLd(
  comments: { body: string | null; author_name: string | null; created_at: string }[],
  defaultAuthor: string,
) {
  return comments
    .filter((c) => c.body?.trim())
    .slice(0, 10)
    .map((c) => ({
      "@type": "Review",
      reviewBody: c.body!.trim(),
      author: { "@type": "Person", name: c.author_name?.trim() || defaultAuthor },
      datePublished: c.created_at.slice(0, 10),
    }));
}
```

Embed as `Restaurant.review[]`. **Rule:** if it's in JSON-LD, the same text must appear in SSR HTML.

### 4D. Dynamic Open Graph

- `app/opengraph-image.tsx` — brand card (home)
- `app/[entity]/[id]/opengraph-image.tsx` — fetch entity; show name + subtitle
- Share URLs with query params → `app/api/og/.../route.ts` returning `ImageResponse`
- Set `export const dynamic = "force-dynamic"` on OG routes that fetch fonts/data at runtime
- Load at least one font for `ImageResponse` (Satori requirement)

In `generateMetadata`, point `openGraph.images` to generated OG URL when not using a real entity photo.

### 4E. Multi-domain canonical

When one codebase serves `app.example.com`, `sub.example.com`, `otherbrand.net`:

1. Pick **one primary URL** per product/locale
2. All aliases emit `<link rel="canonical" href="PRIMARY">`
3. Sitemap on each host lists **only URLs on that host's canonical origin**
4. GSC: separate property per registrable domain (DNS TXT)
5. Naver: register each Korean primary host; HTML file upload OK for subdomains

---

## Step 5 — Environment variables

Instruct user to set in production (never commit values):

```env
NEXT_PUBLIC_SITE_URL=https://your-domain.com
NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION=
NEXT_PUBLIC_NAVER_SITE_VERIFICATION=
```

Codes come from each console after property registration. Agent wires layout metadata only.

---

## Step 6 — Verify (mandatory)

```bash
npm run build
```

After deploy:

```bash
curl -sL "https://YOUR_DOMAIN/" | grep -E 'description|og:description|google-site-verification|naver-site-verification'
curl -sL "https://YOUR_DOMAIN/robots.txt"
curl -sL "https://YOUR_DOMAIN/sitemap.xml" | head -20
curl -sI "https://YOUR_DOMAIN/opengraph-image"
```

Sample detail + FAQ pages. [Rich Results Test](https://search.google.com/test/rich-results) on one detail URL with reviews.

Do not claim success without build output or curl evidence.

---

## Step 7 — User handoff

### Google Search Console
1. Add property (Domain TXT recommended for subdomains)
2. Verify (HTML meta tag or DNS)
3. **색인 생성 → Sitemaps** → primary sitemap URL
4. URL inspection: home + one detail + `/faq`

### Naver Search Advisor
1. Register **https://YOUR_DOMAIN** (NOT http)
2. Verify (meta tag or HTML file for subdomain)
3. **요청 → 사이트맵 제출**
4. URL inspection: description ≤ 80, og matches meta

### AEO/GEO monitoring (week 1)
- GSC → Performance + Page indexing
- Test FAQ/detail URLs in Rich Results Test
- Share a link in Kakao/iMessage — confirm dynamic OG
- Optional: `llms.txt` at site root describing product + key URLs (emerging GEO convention)

---

## Common Mistakes

- FAQ schema ≠ visible page content
- Review schema without SSR review text
- One long description inherited site-wide
- og:description ≠ meta description (Naver fails)
- SPA-only with no crawlable HTML (GEO fails)
- Multiple canonical URLs for same entity across domains
- Hardcode verification tokens in source
- Skip build / skip live verification

---

## Output Format

When finished, provide:

1. **Summary** — SEO + AEO/GEO what was implemented
2. **Files changed** — path + one-line purpose
3. **Env vars** — names only
4. **Console steps** — Google + Naver
5. **Verification** — build + curl + Rich Results (if applicable)
6. **Follow-up** — indexing wait, optional GSC properties for extra domains

---

## Additional Resources

- Code templates: [reference.md](reference.md)
- Trigger examples: [examples.md](examples.md)
