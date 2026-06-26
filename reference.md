# Reference — SEO + AEO + GEO Patterns (Google + Naver)

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

---

## FAQ page (AEO)

### Content + route

```ts
// lib/faq/content.ts
export type FaqItem = { question: string; answer: string };

export const SITE_FAQ: FaqItem[] = [
  {
    question: "What is YourBrand?",
    answer: "Short factual answer visible on the page.",
  },
  // ...
];
```

```tsx
// components/seo/FaqPageView.tsx
import { faqJsonLd } from "@/lib/seo";
import { JsonLd } from "./JsonLd";

export function FaqPageView({ items, title }: { items: FaqItem[]; title: string }) {
  return (
    <>
      <JsonLd data={faqJsonLd(items)} />
      <main>
        <h1>{title}</h1>
        <dl>
          {items.map((item) => (
            <div key={item.question}>
              <dt>{item.question}</dt>
              <dd>{item.answer}</dd>
            </div>
          ))}
        </dl>
      </main>
    </>
  );
}
```

```ts
// lib/seo.ts
export function faqJsonLd(items: { question: string; answer: string }[]) {
  return {
    "@context": "https://schema.org",
    "@type": "FAQPage",
    mainEntity: items.map((item) => ({
      "@type": "Question",
      name: item.question,
      acceptedAnswer: { "@type": "Answer", text: item.answer },
    })),
  };
}
```

Naver meta for `/faq`: unique description ≤ 80 chars.

---

## Organization + Breadcrumb JSON-LD

```ts
export function organizationJsonLd() {
  return {
    "@context": "https://schema.org",
    "@type": "Organization",
    name: SITE_NAME,
    url: SITE_URL,
    logo: `${SITE_URL}/logo.png`,
  };
}

export function breadcrumbJsonLd(
  items: { name: string; href: string }[],
) {
  return {
    "@context": "https://schema.org",
    "@type": "BreadcrumbList",
    itemListElement: items.map((item, i) => ({
      "@type": "ListItem",
      position: i + 1,
      name: item.name,
      item: item.href.startsWith("http") ? item.href : `${SITE_URL}${item.href}`,
    })),
  };
}
```

---

## Entity JSON-LD with reviews (AEO + GEO)

```ts
export function reviewJsonLd(
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

export function restaurantJsonLd(
  place: YourEntityType,
  comments: { body: string | null; author_name: string | null; created_at: string }[] = [],
) {
  return {
    "@context": "https://schema.org",
    "@type": "Restaurant",
    name: place.name,
    url: `${SITE_URL}/items/${place.id}`,
    description: entityLongDescription(place),
    address: { "@type": "PostalAddress", streetAddress: place.address },
    geo: {
      "@type": "GeoCoordinates",
      latitude: place.lat,
      longitude: place.lng,
    },
    ...(place.rating != null && place.review_count != null
      ? {
          aggregateRating: {
            "@type": "AggregateRating",
            ratingValue: place.rating,
            reviewCount: place.review_count,
          },
        }
      : {}),
    review: reviewJsonLd(comments, "Community member"),
  };
}
```

SSR detail page must render the same review bodies in HTML.

---

## Dynamic Open Graph (ImageResponse)

```tsx
// app/items/[id]/opengraph-image.tsx
import { ImageResponse } from "next/og";
import { fetchEntityById } from "@/lib/server";

export const dynamic = "force-dynamic";

export default async function Image({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const entity = await fetchEntityById(id);
  if (!entity) return new ImageResponse(<div>Not found</div>, { width: 1200, height: 630 });

  return new ImageResponse(
    (
      <div style={{ display: "flex", width: "100%", height: "100%", background: "#111", color: "#fff", padding: 48 }}>
        <div style={{ fontSize: 48, fontWeight: 700 }}>{entity.name}</div>
      </div>
    ),
    { width: 1200, height: 630 },
  );
}
```

Share links with query params:

```ts
// app/api/og/list/route.ts
import { ImageResponse } from "next/og";

export const dynamic = "force-dynamic";

export async function GET(req: Request) {
  const token = new URL(req.url).searchParams.get("token");
  // fetch list title by token...
  return new ImageResponse(/* card JSX */, { width: 1200, height: 630 });
}
```

Wire in metadata:

```ts
openGraph: {
  images: [{ url: `/items/${id}/opengraph-image`, width: 1200, height: 630 }],
},
// shared list:
openGraph: {
  images: [{ url: `/api/og/list?token=${token}`, width: 1200, height: 630 }],
},
```

---

## Multi-domain canonical + host-aware sitemap

```ts
// lib/domains.ts
export function primaryOrigin(host: string): string {
  if (host.includes("sub.example.com")) return "https://sub.example.com";
  return "https://example.com";
}

export function entityPublicPath(id: string): string {
  return `/items/${id}`;
}
```

```ts
// app/sitemap.ts — use headers() to pick origin
import { headers } from "next/headers";
import { primaryOrigin } from "@/lib/domains";

export default async function sitemap() {
  const host = (await headers()).get("host") ?? "example.com";
  const origin = primaryOrigin(host);
  // emit URLs only on `origin`
}
```

Each alias domain: `<link rel="canonical" href="{primaryOrigin}{path}">`.

---

## Optional: llms.txt (GEO)

Plain text at `/public/llms.txt`:

```txt
# YourBrand
> Curated map of X in City Y.

## Key pages
- https://example.com/faq
- https://example.com/items/{id}
```
