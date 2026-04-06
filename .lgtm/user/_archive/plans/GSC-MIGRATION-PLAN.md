# Google Search Console Migration Plan

## Context

silverthreadlabs.com currently points to an **old site** (also built on the Bloggen SEO starter, hosted on Vercel). This **new site** (this project) is a separate Vercel deployment with no custom domain attached yet. Both are live concurrently.

The URL structures are **completely different** between old and new. This makes the redirect map the single most critical piece of the migration -- every old indexed URL that isn't redirected becomes a 404.

GSC is verified via **DNS TXT record**, which means verification survives the domain reassignment automatically. No code changes needed for GSC verification.

Since both sites are on Vercel, migration day is just reassigning the custom domain from the old project to the new one in the Vercel dashboard. No DNS changes required.

---

## What I can and cannot do

**Can do:**
- Crawl the old site's live sitemap.xml to capture its URL inventory (it's still live)
- Build a redirect map: old URLs -> best-matching new URLs (or 410 for removed content)
- Implement 301 redirects in `next.config.ts`
- Fix hardcoded domain references in `lib/config/site.ts`
- Write a post-migration redirect verification script

**Cannot do:**
- Reassign the domain in Vercel dashboard (manual)
- Access GSC data (no API credentials)
- Submit sitemaps in GSC UI (manual)

---

## Migration Path

### Phase A: Pre-Migration (before domain reassignment)

**A1. Capture old site URL inventory**
- Fetch the old site's `sitemap.xml` now (before it disappears)
- Also export from GSC: Performance > Pages (all URLs with impressions, last 16 months)
- Combine and deduplicate into a master list

**A2. Build redirect map** (Claude Code + your review)
- Compare every old URL against the new site's ~166 URLs
- Match by content intent (e.g., old `/our-services/x` -> new `/services/x`)
- Output: CSV with `old_path, new_path, status` (301 or 410)
- You review before implementation

**A3. Implement redirects in next.config.ts**
- Add `redirects()` function with the approved map
- 301 for content that has a new home
- For intentionally removed content: middleware returning 410 Gone

**A4. Fix hardcoded domains in site.ts**
- `lib/config/site.ts:34` -- `author.url`: hardcoded `https://www.silverthreadlabs.com` -> `getURL()`
- `lib/config/site.ts:35` -- `author.logo`: same fix
- `lib/config/site.ts:48` -- `social.sameAs[0]`: same fix

**A5. Set env var** (manual, Vercel dashboard)
- Set `NEXT_PUBLIC_SITE_URL=https://www.silverthreadlabs.com` on the **new** Vercel project
- This drives all canonical URLs, sitemaps, OG images, JSON-LD schemas

### Phase B: Migration Day

Since both are on Vercel, this is simple:

1. Pre-flight: verify redirects work on the new site's Vercel preview URL
2. In Vercel dashboard: remove `silverthreadlabs.com` from old project
3. In Vercel dashboard: add `silverthreadlabs.com` + `www.silverthreadlabs.com` to new project
4. Vercel handles SSL automatically -- no DNS changes needed
5. Verify: HTTPS works, redirects fire, sitemap.xml returns new content, robots.txt correct
6. GSC verification stays intact (DNS TXT record unchanged)

### Phase C: Post-Migration (days/weeks after)

1. Submit new `sitemap.xml` in GSC (manual)
2. Remove old sitemap from GSC if it had a different filename
3. Monitor GSC Coverage report daily for 2 weeks -- watch for 404 spikes
4. Track Performance report weekly -- expect 10-20% traffic dip for 2-4 weeks, recovery in 2-6 weeks

---

## What happens to indexed pages

| Scenario | Result |
|----------|--------|
| Old URL redirected (301) to new URL | ~99% link equity transfers. Google updates index within days. |
| Old URL returns 404 (no redirect) | Link equity lost. Page drops from index. **Must prevent this.** |
| Old URL returns 410 Gone | Intentional removal. Faster deindexing than 404. |
| New URL not on old site | Discovered via sitemap + crawl. Indexed as new content. |

---

## Key files to modify

| File | Change |
|------|--------|
| `lib/config/site.ts` | Replace 3 hardcoded domain refs with `getURL()` |
| `next.config.ts` | Add `redirects()` with the full redirect map |

---

## Immediate next step

You need to get me the old site's URL list. Two options:

1. **I fetch the old site's sitemap.xml right now** (it's still live) -- gives me the URLs the old site declares
2. **You export from GSC** -- gives me the URLs Google actually indexed (may include pages not in sitemap)

Ideally both, but option 1 I can do immediately.

---

## Verification

After implementation:
- `pnpm build` passes with zero errors
- Old URLs from redirect map return 301 with correct `Location` header
- Removed URLs return 410 Gone
- `sitemap.xml` on preview deployment lists all ~166 new URLs
- `robots.txt` on preview deployment is correct
- No hardcoded `https://www.silverthreadlabs.com` remaining in `site.ts`
