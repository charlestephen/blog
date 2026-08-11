# Cloudflare setup

Canonical site: **`blog.cst.nyc`** (served by GitHub Pages).
The other three subdomains **301-redirect** to it, preserving path + query string.

Four Cloudflare zones are involved: `cst.nyc` (canonical) plus `bkhd.nyc`,
`handhomo.com`, `charlestephen.com` (redirects). Each redirect lives in its **own** zone.

---

## 1. Canonical domain — zone `cst.nyc`

### DNS
| Type  | Name   | Target                     | Proxy |
|-------|--------|----------------------------|-------|
| CNAME | `blog` | `charlestephen.github.io`  | see note |

**Proxy note (important, avoids a TLS chicken-and-egg):**
1. Create the record **DNS-only (grey cloud)** first.
2. In the GitHub repo → **Settings → Pages**, set the custom domain to `blog.cst.nyc`
   and wait until GitHub shows the cert is issued and **Enforce HTTPS** is available (a few min).
3. *(Optional)* Flip the record to **Proxied (orange cloud)** and set the zone's
   **SSL/TLS mode to Full (strict)** to get Cloudflare's CDN/analytics in front of Pages.
   Leaving it DNS-only is also fine — GitHub serves HTTPS either way.

---

## 2. Redirect domains — zones `bkhd.nyc`, `handhomo.com`, `charlestephen.com`

Do this **once per zone**, swapping the hostname each time
(`blog.bkhd.nyc`, `blog.handhomo.com`, `blog.charlestephen.com`).

### DNS (must be **Proxied** so the edge redirect fires)
| Type  | Name   | Target         | Proxy    |
|-------|--------|----------------|----------|
| CNAME | `blog` | `blog.cst.nyc` | Proxied 🟠 |

The target is never actually reached — a proxied record just lets Cloudflare
terminate TLS and run the redirect rule at the edge.

### Redirect rule (Rules → **Redirect Rules** → Create)
- **When incoming requests match:**
  `Hostname` **equals** `blog.bkhd.nyc`   *(← change per zone)*
- **Then... → Dynamic redirect:**
  - Expression: `concat("https://blog.cst.nyc", http.request.uri.path)`
  - Status code: **301**
  - **Preserve query string:** ✅

That sends `https://blog.bkhd.nyc/posts/foo/?x=1` → `https://blog.cst.nyc/posts/foo/?x=1`.

---

## 3. Cloudflare Web Analytics (canonical only)

1. Cloudflare dashboard → **Analytics & Logs → Web Analytics → Add a site** → `blog.cst.nyc`.
2. Copy the **token** from the JS snippet it shows.
3. Paste it into `hugo.toml` → `[params.analytics.cloudflare] token = "..."`.
4. Commit + push. The beacon only loads in the production build (`HUGO_ENVIRONMENT=production`).

> No need to paste Cloudflare's `<script>` by hand — `layouts/partials/extend_head.html`
> emits it from the token.
