---
name: web-seo-google-naver
description: >-
  Implements Google Search Console and Naver Search Advisor SEO for Next.js App
  Router web apps: sitemap, robots, verification meta tags, per-page metadata,
  JSON-LD, and Naver-compliant descriptions (80 chars, og:description match).
  Use when deploying a web service, setting up SEO, Search Console, 서치어드바이저,
  sitemap.xml, robots.txt, openGraph, or when the user asks for Google/Naver SEO.
  Cursor: invoke with @web-seo-google-naver. Claude Code: invoke with /web-seo-google-naver.
---

# Web SEO — Google + Naver (Next.js)

Implement production SEO end-to-end: code, env instructions, build verification, and console handoff. Work autonomously after discovery.

## When to Use

- User deploys a Next.js web app and wants search indexing
- User mentions Google Search Console, Naver Search Advisor (서치어드바이저), sitemap, robots, meta tags, JSON-LD
- User hit Naver URL inspection warnings (description > 80 chars, og:description mismatch)

**When NOT to use:** non-web projects, SEO audit-only with no code changes requested

---

## Done Criteria

### Google
- [ ] `metadata.verification.google` from env
- [ ] `app/sitemap.ts` — all public indexable URLs, https, valid XML
- [ ] `app/robots.ts` — allow `/`, disallow private routes (`/admin`, `/api/`, etc.)
- [ ] Per-page metadata: title, description, canonical, openGraph, twitter
- [ ] JSON-LD on home + detail pages (WebSite, and page-appropriate types)
- [ ] SSR/SSG HTML for indexable routes (not SPA-only shells)
- [ ] Private routes: `noindex`

### Naver
- [ ] `naver-site-verification` in `metadata.verification.other`
- [ ] Every indexable page: `description` === `openGraph.description` === `twitter.description`
- [ ] Every indexable page: description ≤ **80 chars** (CJK-aware: count string length)
- [ ] Every indexable page: **unique** description (no site-wide duplicate)
- [ ] Site registered on **https** (not http)
- [ ] Sitemap submittable at Naver: **요청 → 사이트맵 제출**

### Copy separation (critical)
| Layer | Length | Purpose |
|-------|--------|---------|
| meta / OG / Twitter | ≤ 80 chars | Naver URL inspection |
| JSON-LD / body SEO text | longer OK | Google richness |

Reference: https://searchadvisor.naver.com/guide/markup-content

---

## Step 0 — Discovery

1. Read project rules (`AGENTS.md`, etc.) and `app/layout.tsx`
2. Detect stack: Next.js App Router, deployment target (Vercel, etc.), data source
3. Identify indexable routes vs noindex routes
4. Resolve production domain from env (`NEXT_PUBLIC_SITE_URL`, `VERCEL_URL`) or **ask user**
5. Check existing verification env vars (do not invent codes):
   - `NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION`
   - `NEXT_PUBLIC_NAVER_SITE_VERIFICATION`
6. Run baseline `npm run build` if feasible
7. Brief plan, then implement

**Ask user only when blocked:** missing domain, missing page types, or verification codes needed for live check

---

## Step 1 — Infrastructure

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

Dynamic robots route. Do **not** duplicate with `public/robots.txt`.

### 1E. `app/sitemap.ts`

All public URLs, https, under Naver limits (≤ 50,000 URLs, ≤ 10 MB).

Adapt route names to the project (`/items/[id]`, `/blog/[slug]`, etc.) — do not assume a specific product domain.

---

## Step 2 — Crawlable pages

If the UI is client-heavy (map, dashboard, canvas app):

- Add SSR/SSG routes with real HTML: headings, text, internal links
- `generateMetadata()` per route using naver short copy
- JsonLd component with long description
- Prefer share URLs like `/items/{id}` over `?id=` query params

Match existing project conventions for file paths and components.

---

## Step 3 — Per-page checklist

For each indexable route:

| Check | Rule |
|-------|------|
| description | ≤ 80 chars |
| og:description | === description |
| twitter.description | === description |
| uniqueness | ≠ home and ≠ other pages |
| canonical | absolute https URL |
| title | reflects page content |

Static pages (privacy, terms): short unique descriptions. Admin: noindex.

---

## Step 4 — Environment variables

Instruct user to set in production (never commit values):

```env
NEXT_PUBLIC_SITE_URL=https://your-domain.com
NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION=
NEXT_PUBLIC_NAVER_SITE_VERIFICATION=
```

Codes come from each console after property registration. Agent wires layout metadata only.

---

## Step 5 — Verify (mandatory)

```bash
npm run build
```

After deploy (or local prod if available):

```bash
curl -sL "https://YOUR_DOMAIN/" | grep -E 'description|og:description|google-site-verification|naver-site-verification'
curl -sL "https://YOUR_DOMAIN/robots.txt"
curl -sL "https://YOUR_DOMAIN/sitemap.xml" | head -20
```

Sample 1–2 detail pages. Confirm description === og:description and length ≤ 80.

Do not claim success without build output or curl evidence.

---

## Step 6 — User handoff

### Google Search Console
1. Add property: `https://YOUR_DOMAIN`
2. Verify (HTML meta tag — already in layout)
3. **색인 생성 → Sitemaps** → `https://YOUR_DOMAIN/sitemap.xml`
4. URL inspection: home + one detail page

### Naver Search Advisor
1. Register **https://YOUR_DOMAIN** (NOT http)
2. Verify HTML meta tag (`naver-site-verification`)
3. **요청 → 사이트맵 제출** → `https://YOUR_DOMAIN/sitemap.xml`
4. URL inspection: confirm no description > 80 chars, og matches meta

---

## Common Mistakes

- One long description inherited site-wide
- og:description ≠ meta description
- Naver registered on http
- SPA-only with no crawlable HTML
- Long copy in meta tags (Naver fails)
- Skip build / skip live verification
- Hardcode verification tokens in source

---

## Output Format

When finished, provide:

1. **Summary** — what was implemented
2. **Files changed** — path + one-line purpose
3. **Env vars** — names only; user fills values in hosting dashboard
4. **Console steps** — Google + Naver (Korean menu labels for Naver)
5. **Verification** — build result + curl samples (redact any secrets)
6. **Follow-up** — day-1 / week-1 monitoring checklist

---

## Additional Resources

- Code templates and metadata patterns: [reference.md](reference.md)
- Example trigger message: [examples.md](examples.md)
