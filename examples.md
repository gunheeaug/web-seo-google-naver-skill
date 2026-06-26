# Examples — Trigger Messages

## Cursor — Basic (SEO + Naver)

```
@web-seo-aeo-geo-google-naver

Next.js 앱 Google이랑 네이버 SEO 설정해줘.
도메인: https://myapp.com
```

## Claude Code — Basic

```
/web-seo-aeo-geo-google-naver

Next.js 앱 Google이랑 네이버 SEO 설정해줘.
도메인: https://myapp.com
```

## Codex — Basic

```
$web-seo-aeo-geo-google-naver

Next.js 앱 Google이랑 네이버 SEO 설정해줘.
도메인: https://myapp.com
```

## AEO + GEO (FAQ, reviews, dynamic OG)

```
@web-seo-aeo-geo-google-naver

장소 상세 SSR 페이지, /faq FAQPage JSON-LD, 리뷰 Review 스키마, opengraph-image 동적 OG까지 AEO/GEO 레이어 추가해줘.
도메인: https://myapp.com
```

## Multi-domain canonical

```
@web-seo-aeo-geo-google-naver

mint.example.net이랑 example.com/mint 둘 다 서는데 canonical 하나로 정리하고 호스트별 sitemap 맞춰줘.
```

## Natural prompt (both IDEs — auto-loads skill)

```
배포는 했는데 검색이랑 AI 답변에 안 잡혀. Google Search Console, 네이버 서치어드바이저, FAQ/리뷰 스키마까지 설정해줘.
```

## Naver URL inspection fix

```
네이버 URL 검사에서 description 80자 초과랑 og:description 불일치 경고 떠. 고쳐줘.
```

## SPA / map app

```
지도 앱인데 크롤러용 SSR 장소 페이지, FAQ, SEO/AEO/GEO 메타데이터 추가해줘. Google + 네이버 둘 다.
```

## What the agent should ask back (only if missing)

- Production domain (`https://...`)
- Which routes should be indexed vs noindex
- Whether verification env vars are already set in hosting dashboard
- Primary canonical when multiple domains exist

## What the agent should NOT ask for unnecessarily

- Pre-filled verification token values (user gets these from consoles)
- Product-specific business logic unrelated to SEO/AEO/GEO
