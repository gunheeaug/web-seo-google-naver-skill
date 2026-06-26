# Agent Skill: Web SEO + AEO + GEO (Google + Naver)

**Cursor, Claude Code & Codex skill** for Next.js App Router — classic search indexing (SEO), Answer Engine Optimization (AEO), and Generative Engine Optimization (GEO) end-to-end.

바이브코딩으로 웹 배포한 뒤, Google·네이버 검색 등록 + FAQ/리뷰 스키마 + 동적 OG까지 AI가 코드 작성 → 빌드 검증 → 콘솔 체크리스트까지 안내하게 하는 스킬.

---

## What it does

### SEO (Google + Naver)
- `sitemap.ts`, `robots.ts`, verification meta tags
- Per-page metadata (title, description, canonical, OG, Twitter)
- **Naver**: description ≤ 80 chars, `og:description` === `description`, unique per page
- **Google**: JSON-LD, Search Console sitemap
- SSR crawlable landing pages for SPA/map apps

### AEO + GEO
- `/faq` with visible Q&A + `FAQPage` JSON-LD
- Entity detail SSR pages with `Restaurant`/`LocalBusiness` + **Review** schema from real UGC
- `Organization`, `BreadcrumbList` JSON-LD
- Dynamic OG (`opengraph-image.tsx`, `/api/og/*`)
- Multi-domain canonical + host-aware sitemaps

Build + curl verification before claiming done. No project-specific secrets — works on any Next.js web app.

---

## Install

Same repo, different folder depending on your IDE:

### Cursor

**Personal (all projects):**
```bash
git clone https://github.com/gunheeaug/web-seo-aeo-geo-google-naver-skill.git ~/.cursor/skills/web-seo-aeo-geo-google-naver
```

**Project (one repo):**
```bash
mkdir -p .cursor/skills
git clone https://github.com/gunheeaug/web-seo-aeo-geo-google-naver-skill.git .cursor/skills/web-seo-aeo-geo-google-naver
```

Restart Cursor if the skill does not appear.

### Claude Code

**Personal (all projects):**
```bash
git clone https://github.com/gunheeaug/web-seo-aeo-geo-google-naver-skill.git ~/.claude/skills/web-seo-aeo-geo-google-naver
```

**Project (one repo):**
```bash
mkdir -p .claude/skills
git clone https://github.com/gunheeaug/web-seo-aeo-geo-google-naver-skill.git .claude/skills/web-seo-aeo-geo-google-naver
```

Skills are picked up automatically. If you add the folder mid-session, run `/reload-skills` or restart Claude Code.

### Codex

**Personal (all projects):**

```bash
git clone https://github.com/gunheeaug/web-seo-aeo-geo-google-naver-skill.git ~/.codex/skills/web-seo-aeo-geo-google-naver
```

Or use Codex's built-in installer (repo root is the skill):

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo gunheeaug/web-seo-aeo-geo-google-naver-skill \
  --path . \
  --name web-seo-aeo-geo-google-naver
```

Restart Codex after install.

**Migrating from `web-seo-google-naver`:** remove the old folder and clone again with the new name above.

---

## Usage

### Cursor
```
@web-seo-aeo-geo-google-naver

Next.js 앱 SEO + AEO/GEO 설정해줘.
도메인: https://myapp.com
```

### Claude Code
```
/web-seo-aeo-geo-google-naver

Next.js 앱 SEO + AEO/GEO 설정해줘.
도메인: https://myapp.com
```

### Codex
```
$web-seo-aeo-geo-google-naver

Next.js 앱 SEO + AEO/GEO 설정해줘.
도메인: https://myapp.com
```

Or ask naturally — the skill loads when you mention SEO, AEO, GEO, Search Console, or 서치어드바이저.

More examples: [examples.md](examples.md)

---

## Cursor vs Claude Code vs Codex

| | Cursor | Claude Code | Codex |
|---|--------|-------------|-------|
| Personal path | `~/.cursor/skills/web-seo-aeo-geo-google-naver/` | `~/.claude/skills/web-seo-aeo-geo-google-naver/` | `~/.codex/skills/web-seo-aeo-geo-google-naver/` |
| Project path | `.cursor/skills/web-seo-aeo-geo-google-naver/` | `.claude/skills/web-seo-aeo-geo-google-naver/` | — (use personal path) |
| Invoke | `@web-seo-aeo-geo-google-naver` | `/web-seo-aeo-geo-google-naver` | `$web-seo-aeo-geo-google-naver` |
| Skill file | `SKILL.md` | `SKILL.md` (same) | `SKILL.md` (same) |
| After install | Restart Cursor if needed | `/reload-skills` or restart | Restart Codex |

---

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Agent workflow and done criteria (SEO + AEO + GEO) |
| `reference.md` | Code templates (metadata, sitemap, JSON-LD, OG, multi-domain) |
| `examples.md` | Trigger message examples |

---

## Google vs Naver (quick reference)

| | Google | Naver |
|---|--------|-------|
| Console | [Search Console](https://search.google.com/search-console) | [서치어드바이저](https://searchadvisor.naver.com) |
| Sitemap | 색인 생성 → Sitemaps | 요청 → 사이트맵 제출 |
| Description | ~160 chars, flexible | ≤ 80 chars, unique per page |
| og:description | flexible | **must equal meta description** |
| Register URL | https preferred | **https required** |

Naver markup guide: https://searchadvisor.naver.com/guide/markup-content

---

## Env vars (hosting dashboard — never commit values)

```env
NEXT_PUBLIC_SITE_URL=https://your-domain.com
NEXT_PUBLIC_GOOGLE_SITE_VERIFICATION=
NEXT_PUBLIC_NAVER_SITE_VERIFICATION=
```

---

## License

MIT — use, copy, and share freely.
