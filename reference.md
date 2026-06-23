# Reference — Google + Naver SEO Patterns

Generic templates. Replace placeholders with project-specific names, routes, and copy.

## Placeholders

| Placeholder | Meaning |
|-------------|---------|
| `YOUR_DOMAIN` | Production hostname, e.g. `example.com` |
| `SITE_URL` | `https://YOUR_DOMAIN` |
| `SITE_NAME` | Product/brand display name |
| `/items/[id]` | Example detail route — use actual routes |

---

## `lib/naver-markup.ts` skeleton

```ts
import type { YourEntityType } from "./types";

export const NAVER_DESC_MAX = 80;

export function naverClamp(text: string, max = NAVER_DESC_MAX): string {
  const trimmed = text.trim().replace(/\s+/g, " ");
  if (trimmed.length <= max) return trimmed;
  return `${trimmed.slice(0, max - 1).trimEnd()}…`;
}

export const NAVER_HOME_TITLE = "YourBrand - one line tagline";
export const NAVER_HOME_DESCRIPTION = naverClamp(
  "Short unique home description under 80 characters.",
);

export function naverEntityTitle(entity: YourEntityType): string {
  return naverClamp(`${entity.name} · Category · Location`, 60);
}

export function naverEntityDescription(entity: YourEntityType): string {
  const desc = `${entity.name} — Location. Short unique sentence.`;
  return naverClamp(desc);
}
```

---

## `lib/seo.ts` metadata builder

```ts
import type { Metadata } from "next";
import { naverEntityDescription, naverEntityTitle } from "./naver-markup";

export const SITE_URL =
  process.env.NEXT_PUBLIC_SITE_URL ?? "https://example.com";
export const SITE_NAME = "YourBrand";

export function entityMetadata(entity: YourEntityType): Metadata {
  const title = naverEntityTitle(entity);
  const description = naverEntityDescription(entity);
  const canonical = `${SITE_URL}/items/${entity.id}`;

  return {
    title: { absolute: title },
    description,
    alternates: { canonical },
    openGraph: {
      type: "website",
      locale: "ko_KR",
      url: canonical,
      siteName: SITE_NAME,
      title,
      description,
    },
    twitter: {
      card: "summary_large_image",
      title,
      description,
    },
  };
}

/** Long copy for JSON-LD only — not for meta tags */
export function entityLongDescription(entity: YourEntityType): string {
  return `Longer description with ratings, address, keywords — OK for schema.org.`;
}
```

---

## `app/layout.tsx` verification

```ts
import type { Metadata } from "next";
import { NAVER_HOME_DESCRIPTION, NAVER_HOME_TITLE } from "@/lib/naver-markup";
import { SITE_NAME, SITE_URL } from "@/lib/seo";

const siteVerification: Metadata["verification"] = {
  other: {},
};

if (process.env.NEXT_PUBLIC_NAVER_SITE_VERIFICATION) {
  siteVerification.other!["naver-site-verification"] =
    process.env.NEXT_PUBLIC_NAVER_SITE_VERIFICATION;
}
if (process.env.NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION) {
  siteVerification.google = process.env.NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION;
}

export const metadata: Metadata = {
  metadataBase: new URL(SITE_URL),
  title: { default: NAVER_HOME_TITLE, template: `%s | ${SITE_NAME}` },
  description: NAVER_HOME_DESCRIPTION,
  openGraph: {
    type: "website",
    locale: "ko_KR",
    url: SITE_URL,
    siteName: SITE_NAME,
    title: NAVER_HOME_TITLE,
    description: NAVER_HOME_DESCRIPTION,
  },
  twitter: {
    card: "summary_large_image",
    title: NAVER_HOME_TITLE,
    description: NAVER_HOME_DESCRIPTION,
  },
  verification: siteVerification,
};
```

---

## `app/robots.ts`

```ts
import type { MetadataRoute } from "next";
import { SITE_URL } from "@/lib/seo";

export default function robots(): MetadataRoute.Robots {
  return {
    rules: [
      {
        userAgent: "*",
        allow: "/",
        disallow: ["/admin", "/api/"],
      },
    ],
    sitemap: `${SITE_URL}/sitemap.xml`,
  };
}
```

Adjust `disallow` to match private routes in the project.

---

## `app/sitemap.ts` pattern

```ts
import type { MetadataRoute } from "next";
import { SITE_URL } from "@/lib/seo";

export default async function sitemap(): Promise<MetadataRoute.Sitemap> {
  const staticRoutes: MetadataRoute.Sitemap = [
    { url: SITE_URL, lastModified: new Date(), changeFrequency: "daily", priority: 1 },
    { url: `${SITE_URL}/about`, lastModified: new Date(), changeFrequency: "monthly", priority: 0.5 },
  ];

  // Fetch dynamic IDs from DB/API
  const entities = await fetchAllPublicEntities();
  const dynamicRoutes: MetadataRoute.Sitemap = entities.map((e) => ({
    url: `${SITE_URL}/items/${e.id}`,
    lastModified: e.updated_at ?? new Date(),
    changeFrequency: "weekly",
    priority: 0.7,
  }));

  return [...staticRoutes, ...dynamicRoutes];
}
```

If URL count may exceed 50,000, split into sitemap index files per Naver limits.

---

## JSON-LD component

```tsx
export function JsonLd({ data }: { data: Record<string, unknown> }) {
  return (
    <script
      type="application/ld+json"
      dangerouslySetInnerHTML={{ __html: JSON.stringify(data) }}
    />
  );
}
```

Use long `description` in JSON-LD; keep meta/OG short.

---

## Admin noindex

```ts
// app/admin/layout.tsx
export const metadata: Metadata = {
  robots: { index: false, follow: false },
};
```

---

## Google vs Naver quick reference

| Task | Google | Naver |
|------|--------|-------|
| Console | Search Console | 서치어드바이저 |
| Verify | HTML meta / DNS | HTML meta tag |
| Sitemap menu | 색인 생성 → Sitemaps | 요청 → 사이트맵 제출 |
| Description | ~160 chars, flexible | ≤ 80 chars, unique per page |
| og:description | flexible | must equal meta description |
| Register URL | https preferred | **https required** |
