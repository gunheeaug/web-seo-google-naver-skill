# Agent Skill: Web SEO (Google + Naver)

**Cursor & Claude Code skill** for Next.js App Router — Google Search Console + Naver Search Advisor SEO end-to-end.

바이브코딩으로 웹 배포한 뒤, Google·네이버 검색 등록까지 AI가 코드 작성 → 빌드 검증 → 콘솔 체크리스트까지 안내하게 하는 스킬.

---

## What it does

- `sitemap.ts`, `robots.ts`, verification meta tags
- Per-page metadata (title, description, canonical, OG, Twitter)
- **Naver**: description ≤ 80 chars, `og:description` === `description`, unique per page
- **Google**: JSON-LD, longer schema descriptions, Search Console sitemap
- SSR crawlable landing pages for SPA/map apps
- Build + curl verification before claiming done

No project-specific secrets, domains, or product names — works on any Next.js web app.

---

## Install

Same repo, different folder depending on your IDE:

### Cursor

**Personal (all projects):**
```bash
git clone https://github.com/gunheeaug/cursor-skill-web-seo-google-naver.git ~/.cursor/skills/web-seo-google-naver
```

**Project (one repo):**
```bash
mkdir -p .cursor/skills
git clone https://github.com/gunheeaug/cursor-skill-web-seo-google-naver.git .cursor/skills/web-seo-google-naver
```

Restart Cursor if the skill does not appear.

### Claude Code

**Personal (all projects):**
```bash
git clone https://github.com/gunheeaug/cursor-skill-web-seo-google-naver.git ~/.claude/skills/web-seo-google-naver
```

**Project (one repo):**
```bash
mkdir -p .claude/skills
git clone https://github.com/gunheeaug/cursor-skill-web-seo-google-naver.git .claude/skills/web-seo-google-naver
```

Skills are picked up automatically. If you add the folder mid-session, run `/reload-skills` or restart Claude Code.

---

## Usage

### Cursor
```
@web-seo-google-naver

Next.js 앱 Google이랑 네이버 SEO 설정해줘.
도메인: https://myapp.com
```

### Claude Code
```
/web-seo-google-naver

Next.js 앱 Google이랑 네이버 SEO 설정해줘.
도메인: https://myapp.com
```

Or just ask naturally — Claude loads the skill when you mention SEO, Search Console, or 서치어드바이저.

More examples: [examples.md](examples.md)

---

## Cursor vs Claude Code

| | Cursor | Claude Code |
|---|--------|-------------|
| Personal path | `~/.cursor/skills/web-seo-google-naver/` | `~/.claude/skills/web-seo-google-naver/` |
| Project path | `.cursor/skills/web-seo-google-naver/` | `.claude/skills/web-seo-google-naver/` |
| Invoke | `@web-seo-google-naver` | `/web-seo-google-naver` |
| Skill file | `SKILL.md` | `SKILL.md` (same) |

---

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Agent workflow and done criteria |
| `reference.md` | Code templates (metadata, sitemap, JSON-LD) |
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
