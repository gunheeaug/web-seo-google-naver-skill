# Examples — Trigger Messages

## Cursor — Basic

```
@web-seo-google-naver

Next.js 앱 Google이랑 네이버 SEO 설정해줘.
도메인: https://myapp.com
```

## Claude Code — Basic

```
/web-seo-google-naver

Next.js 앱 Google이랑 네이버 SEO 설정해줘.
도메인: https://myapp.com
```

## Natural prompt (both IDEs — auto-loads skill)

```
배포는 했는데 검색에 안 잡혀. Google Search Console이랑 네이버 서치어드바이저 SEO 코드랑 sitemap 설정해줘.
```

## Naver URL inspection fix

```
네이버 URL 검사에서 description 80자 초과랑 og:description 불일치 경고 떠. 고쳐줘.
```

## SPA / map app

```
지도 앱인데 크롤러용 SSR 랜딩 페이지랑 SEO 메타데이터 추가해줘. Google + 네이버 둘 다.
```

## What the agent should ask back (only if missing)

- Production domain (`https://...`)
- Which routes should be indexed vs noindex
- Whether verification env vars are already set in hosting dashboard

## What the agent should NOT ask for unnecessarily

- Pre-filled verification token values (user gets these from consoles)
- Product-specific business logic unrelated to SEO
