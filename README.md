# robodor.com — Case Study

A six-locale B2B industrial product catalog and CMS, designed and built for
**[Robodor](https://www.robodor.com)** — a Turkish manufacturer of motor drives,
encoders and sliding-door drivers serving customers across Türkiye and Europe.

> This repository is a **case study only**. It contains no source code from
> the production application. The live site is at
> **[https://www.robodor.com](https://www.robodor.com)**.
>
> The production repository and commit history are private because they contain
> company-owned source code, admin flows, deployment configuration and product
> details. This public repo is the sanitized portfolio layer: screenshots from
> the live public site, architecture notes, trade-offs, and enough technical
> detail to review the work without exposing Robodor's implementation.

---

## TR · Özet

Robodor için endüstriyel kapı kontrol sistemleri kataloğu ve admin CMS'i
tasarlayıp geliştirdim. Site **6 dilde** (TR varsayılan, EN, FR, DE, IT, ES)
B2B müşterilere ulaşıyor; içerik üretimi tamamen şirket içi admin panelinden
yönetiliyor.

**Stack:** Next.js 16 (App Router) · React 19 · TypeScript (no `any`) ·
Prisma + Neon Postgres · NextAuth v5 + Google OAuth · Puck CMS ·
Tailwind + shadcn/ui · AWS S3 · Resend · Google Analytics 4 · Vercel.

**Öne çıkan iş kalemleri:**

- Kendi yazdığım admin panel: ürün CRUD, haberler, referanslar, form
  konfigürasyonları, çeviri yönetimi, yasal sayfa editörü — hepsi 6 dil
  destekli, locale-tab UX'i ile.
- Veritabanı-öncelikli içerik mimarisi (`DB → hardcoded fallback`) —
  admin DB'yi düzenlerken kod tabanı her zaman çalışan bir varsayılana
  sahip; deploy ile içerik birbirinden bağımsız.
- 6 lokale için tam SEO çıktısı: `metadataBase`, locale başına
  `canonical`, `hreflang` + `x-default`, lokalize meta description'lar,
  sitemap, eski IIS rotalarından 301 redirect.
- KVKK/GDPR-oriented legal content management for Gizlilik / Çerez / KVKK
  Aydınlatma metinleri — DB-edit'lenebilir (admin'den anında düzenlenir,
  `force-dynamic` ile yayına yansır).
- Telefon validasyonu: 100+ ülke için ülke kodu seçici, ülkeye özel
  uzunluk ve format kontrolü, `Intl.DisplayNames` ile locale'a göre ülke
  ad lokalizasyonu.
- Erişilebilirlik: 6 dilde "Skip to content", aria-pressed pause kontrolü
  olan otomatik dönen hero, semantic h1 hierarchy, WCAG 2.2.2 uyumu.
- Audit-sürücülü kalite turları: 6 ardışık dış audit raporu sonrası
  çıkan ~50 kalem (UI overflow, çeviri kalitesi, h1 tutarlılığı, decimal
  separator, locale leak, vs.) private/deploy repo commit zincirinde takip
  edildi.

---

## EN · Summary

Designed and built a B2B product catalog and CMS for Robodor, a Turkish
manufacturer of door-control electronics. The site reaches customers across
**six locales** (TR default; EN, FR, DE, IT, ES); all content is produced
in-house via the custom admin panel.

**Stack:** Next.js 16 (App Router) · React 19 · TypeScript (no `any`) ·
Prisma + Neon Postgres · NextAuth v5 + Google OAuth · Puck CMS ·
Tailwind + shadcn/ui · AWS S3 · Resend · Google Analytics 4 · Vercel.

**Highlights:**

- Custom-built admin panel: product CRUD, news, references, form configs,
  translation manager, legal-page editor — every surface localized in six
  languages via a locale-tabbed editor pattern I designed.
- DB-first content with hardcoded fallback: the admin can edit content
  while the codebase always has a known-good default. Code deploys and
  content edits are decoupled.
- Complete SEO surface for six locales: `metadataBase`, per-locale
  `canonical`, `hreflang` map with `x-default`, localized meta
  descriptions, sitemap, 301 redirects from legacy IIS routes.
- KVKK/GDPR-oriented legal content management for Privacy / Cookies / KVKK
  information notices — DB-editable from the admin, served via `force-dynamic`
  so edits go live without redeploy.
- Phone validation: country-code picker for 100+ countries, per-country
  length and format rules, `Intl.DisplayNames` for locale-aware country
  names.
- Accessibility: six-locale "Skip to content", aria-pressed pause control
  on the rotating hero, semantic h1 hierarchy, WCAG 2.2.2-compliant
  motion controls.
- Iterative audit-driven hardening: ~50 issues across 6 external review
  rounds (UI overflow, translation quality, h1 consistency, decimal
  separators, hardcoded TR leaks, etc.) tracked and shipped in the private
  production/deploy repository.

---

## Public Evidence

This repo is intentionally not a source mirror. The strongest public proof is:

- **Live product:** [robodor.com](https://www.robodor.com) is the production
  site.
- **Public screenshots:** wide desktop captures from the live site are in
  [`screenshots/`](screenshots/).
- **Architecture notes:** this README documents the content model, admin
  workflows, SEO/i18n surface, deployment shape, and hardening decisions.
- **Disclosure boundary:** no admin screenshots, database schema dumps, secrets,
  customer-only files, upload paths, or proprietary business logic are included.

---

## Architecture in a Few Diagrams

```
                                 ┌──────────────────┐
                                 │   Vercel (FRA1)  │
   ┌───────────────┐             │ Fluid Compute    │             ┌─────────────────┐
   │  robodor.com  │ ───────────▶│ Next.js App      │────────────▶│ Neon Postgres   │
   │  (apex + www) │             │ Router           │  Prisma     │ (eu-central-1)  │
   └───────────────┘             │                  │             └─────────────────┘
                                 │  RSC + force-    │
                                 │  dynamic admin   │             ┌─────────────────┐
                                 │                  │────────────▶│ AWS S3 uploads  │
                                 └────────┬─────────┘ Presigned   │ (us-east-1)     │
                                          │                       └─────────────────┘
                              ┌───────────┴───────────┐
                              ▼                       ▼
                       ┌─────────────┐         ┌─────────────┐
                       │  Resend     │         │  Google     │
                       │  (email)    │         │  OAuth +    │
                       │             │         │  GA4        │
                       └─────────────┘         └─────────────┘
```

**Why Vercel + Neon + S3 over a single managed stack?** Vercel for the
edge cache and instant rollbacks; Neon for branchable Postgres with
zero-downtime migrations on the EU side; S3 for the 25 MB datasheet PDFs
and 4.9 GB software binaries that don't belong in a database. Each piece
swappable, each piece priced for what it does.

---

## Notable Engineering Decisions

### Six-locale content model with no per-locale duplication

Every translatable record has its labels stored as a single JSON column
(`{ tr, en, fr, de, it, es }`) rather than split into six separate rows.
This means the schema doesn't grow with each new language, the admin
edits in one place, and the renderer trivially picks the right slot at
read time with EN fallback.

The same pattern applies to product specs (the spec values themselves
needed full localization too, not just the labels — early iterations
shared a single `value` field across locales, which leaked Turkish into
French / Italian product cards. Migrated to per-locale `valueXx` fields
mid-project; documented in the commit history).

### `force-dynamic` admin-driven pages

Pages like product detail, legal docs and news read from the DB at every
request rather than relying on Next's static cache. Trade-off: a few
hundred extra ms on cold cache. Win: when the admin edits a product spec
in /admin, the change is live on the next page load with no rebuild, no
revalidation dance.

### Two-remote git pipeline

The repo lives at the developer's personal GitHub mirror and at the
company's organisation account. Vercel watches only the company remote.
Cherry-pick onto the deploy remote keeps personal history clean while
shipping through company-controlled CI. (Lesson learned the hard way:
pushing to the wrong remote left prod stale while the DB had already
migrated. Documented in the project's own CLAUDE.md so future contributors
don't repeat it.)

### Audit-driven hardening, not big-bang launch

Instead of one massive QA pass before launch, the project shipped an
initial usable version and absorbed five rounds of external audit review
afterwards. Each round produced a focused commit (`fix(audit-N): ...`)
that addressed only that round's findings. The commit log reads like a
debug diary — and that's a feature, not a bug, when an SRE-minded
reviewer comes back to ask "how did you handle X".

---

## Featured Admin Capabilities

### Locale-tabbed editors everywhere

Admin needs to edit one record across six languages. Instead of six
separate form views, every multilingual record (product, legal doc,
form-config field, translation row) is edited through a single editor
component with a six-locale tab strip:

- Tab labels show `TR · EN · FR · DE · IT · ES`
- Each tab has a green dot indicator when its content is filled
- A "Copy from EN" helper for any non-EN tab speeds the first pass
- The same pattern is reused everywhere — once the admin learns one
  editor, they know them all.

### Product specs editor

Each spec row is a card with: header preview (EN label / EN value),
locale tabs, and side-by-side label + value inputs per tab. Earlier
iterations crammed all 12 fields into one row; the user audit said it
was unreadable. The redesign showed only the active locale's two fields,
making the editor 6× simpler at the moment of edit.

### Live legal page editor

Privacy, Cookies and KVKK notices are stored as structured JSON in the
DB (`{ title, intro[], sections[] }` per locale) rather than as a free
markdown blob. Admin can add / remove / reorder sections, paragraphs and
bullets, all per-locale, without touching code. The page reads from DB
first and falls back to a hardcoded source-of-truth if no DB row exists,
so the site is never empty.

### Translation manager

Free-text taxonomy values (`categoryType`, `application`, `phase`) are
stored as keys with localized labels. Admin can add new entries and
translate them across six locales. Combined with the dynamic filter
URL pattern (`?application=high-speed`), this means new product
categories can ship without code changes.

---

## SEO & Internationalization

- `metadataBase: https://www.robodor.com` site-wide; sitemap, robots.txt
  and canonical URLs all use the same host (no www-vs-apex split).
- Each public page emits `alternates.canonical` pointing at the current
  locale's URL plus `alternates.languages` with all six locale variants
  and `x-default` → `/en/...`.
- Country → locale mapping in middleware covers ~89 countries (TR;
  Italy / San Marino / Vatican; Spain + 19 LATAM; France + Francophone
  Africa; DACH; Anglosphere) with an Accept-Language fallback to EN.
- 301 redirects from the legacy IIS site preserve SEO equity:
  `/tr-TR/Index → /tr`, `/en-US/PrivacyNotice → /en/gizlilik`,
  `/tr-TR/ProductListPage?catId=N → /tr/urunler`, etc.
- Hardcoded Turkish product labels that appeared in EN / DE pages were caught
  and routed through `next-intl` — flagged via diff-of-extracted-text-per-locale
  on every deploy.

---

## Accessibility & Web Vitals

- Six-locale "Skip to content" link — properly localized, not buried.
- Hero auto-rotates between product variants; a Pause / Play toggle
  with `aria-pressed` exposes the rotation state to keyboard and screen
  readers, supporting WCAG 2.2.2-style pause/stop/hide expectations.
- Single semantic `h1` per page, with the slider product names dropped
  to `h2`.
- All `<Image>` usages have explicit `alt` text — verified by an
  AST-walking audit script.
- Fonts loaded via `next/font` (no FOUT), images via `next/image`
  (responsive `srcSet`, lazy by default).

---

## Screenshots

Screenshots are visual proof of the live work. They are captured at
**1920 × 1080** from public pages on www.robodor.com only.

| Surface | File |
|---|---|
| Home hero | [`screenshots/01-home-hero-desktop.png`](screenshots/01-home-hero-desktop.png) |
| Product listing with active category filter | [`screenshots/02-product-list-desktop.png`](screenshots/02-product-list-desktop.png) |
| German product detail page | [`screenshots/03-product-detail-desktop.png`](screenshots/03-product-detail-desktop.png) |
| References page | [`screenshots/04-references-desktop.png`](screenshots/04-references-desktop.png) |

---

## Live

- **Production:** https://www.robodor.com — six locales, real product catalog
- **Sample paths to test:**
  - `https://www.robodor.com/tr` — Turkish home
  - `https://www.robodor.com/de/urunler/mr220` — German product detail
  - `https://www.robodor.com/it/urunler` — Italian listing with filter
  - `https://www.robodor.com/fr/iletisim` — French contact form
  - `https://www.robodor.com/en/gizlilik` — English privacy policy

---

## Contact

Built by **Göktürk Göçen** — [github.com/gokturkgocen](https://github.com/gokturkgocen) ·
[gokturk@robodor.com](mailto:gokturk@robodor.com)
