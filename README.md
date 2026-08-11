# The HandyHomo Homelab Blog

Source for **[blog.cst.nyc](https://blog.cst.nyc/)** — a [Hugo](https://gohugo.io/)
static site (theme: [PaperMod](https://github.com/adityatelange/hugo-PaperMod)),
built and deployed to **GitHub Pages** by GitHub Actions on every push to `main`.

## Local development

```bash
git clone --recurse-submodules <repo-url>
cd blog
hugo server -D          # http://localhost:1313, live reload, includes drafts
```

Already cloned without submodules? `git submodule update --init --recursive`.

## New post

```bash
hugo new content posts/my-post.md   # then edit, set draft: false, and push
```

## Publish

Push to `main`. The [`hugo.yml`](.github/workflows/hugo.yml) workflow builds and deploys.

## Setup docs

- [`CLOUDFLARE.md`](CLOUDFLARE.md) — DNS, the 301 redirect domains, and Web Analytics.
- [`COMMENTS.md`](COMMENTS.md) — enabling giscus comments.
