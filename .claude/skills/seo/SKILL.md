---
name: seo
description: Audit or implement SEO on Nuxt public pages — meta tags, Open Graph, structured data, rendering strategy, and Core Web Vitals.
argument-hint: "[audit | page/path/to/page.vue]"
---

The user wants to work with SEO. Action: $ARGUMENTS

## Current public pages

!`find /Users/adel/www/apps/rendevo/apps/frontend/app/pages -name "*.vue" 2>/dev/null | head -30 || echo "No pages found"`

---

## Mode: audit

If $ARGUMENTS is `audit` or empty, audit all public pages against `rules/seo.md`:

For each page found above:
1. Read the file
2. Check every item in the checklist below
3. Report findings with file path and line number

### Checklist

- [ ] `useSeoMeta()` present with `title`, `description`, `ogTitle`, `ogDescription`, `ogImage`
- [ ] `title` under 60 characters
- [ ] `description` under 160 characters
- [ ] Canonical URL defined via `useHead()`
- [ ] Single `<h1>` matching the page title
- [ ] Hierarchical headings — no skipped levels
- [ ] All images use `<NuxtImg>` with `alt`, `width`, `height`
- [ ] Rendering strategy correct (public → SSR/SSG, authenticated → `ssr: false`)
- [ ] JSON-LD structured data present where relevant
- [ ] Page not blocked by `noindex` unintentionally

End with: count of pages fully compliant vs total, and a prioritized list of fixes.

---

## Mode: page (implement SEO on a specific page)

If $ARGUMENTS is a file path, read that page and implement missing SEO:

1. Read the page file in full
2. Identify: what type of content is this? (marketing, blog, product, etc.)
3. Apply `rules/seo.md` — add `useSeoMeta()`, canonical, structured data if relevant
4. Verify rendering strategy matches the page type
5. Output the updated page with all SEO additions

Follow `rules/nuxt-conventions.md` for all Nuxt code written.
