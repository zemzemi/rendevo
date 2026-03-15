---
name: seo
description: SEO conventions for all public-facing pages in apps/frontend/. Applied when building or reviewing Nuxt pages that are crawlable.
---

SEO applies only to **public pages** (marketing, blog, landing pages). Authenticated pages (`definePageMeta({ ssr: false })`) have no SEO concern.

## Meta tags — required on every public page

```ts
useSeoMeta({
  title: 'Page Title | Rendevo',        // max 60 chars
  description: 'Under 160 chars.',
  ogTitle: 'Page Title | Rendevo',
  ogDescription: 'Under 160 chars.',
  ogImage: '/og/page-name.png',
  twitterCard: 'summary_large_image',
})
```

Always add a canonical URL:

```ts
useHead({
  link: [{ rel: 'canonical', href: 'https://rendevo.com/page' }],
})
```

---

## Rendering strategy

| Page type | Strategy |
|---|---|
| Marketing, blog, public content | SSG (`prerender: true`) or SSR |
| Authenticated pages | SPA — `definePageMeta({ ssr: false })` |

Configure in `nuxt.config.ts` or per page with `definePageMeta`.

---

## Images

Always use `<NuxtImg>` — never `<img>`:

```vue
<NuxtImg src="/hero.png" width="1200" height="630" alt="Description" loading="lazy" />
```

- `loading="lazy"` on all images except above-the-fold
- Always provide `width`, `height`, and `alt`

---

## Headings

- Single `<h1>` per page — matches the page title
- Hierarchical: `<h2>` → `<h3>` → `<h4>`, never skip levels

---

## Structured data (JSON-LD)

Use inline `<script type="application/ld+json">` for relevant content types:

```vue
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "{{ title }}",
  "datePublished": "{{ date }}"
}
</script>
```

Use for: Article, Product, FAQ, BreadcrumbList, Organization.

---

## Performance targets

- Lighthouse Performance ≥ 90 on mobile
- Fonts: `font-display: swap`, preload critical fonts via `useHead()`
- Non-critical components: `defineAsyncComponent()`

---

## Robots

Block in `robots.txt`: `/admin`, `/api/`, auth pages, low-value paginated pages.
Never block CSS, JS, or image assets.
