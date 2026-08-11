---
title: "Hello World"
date: 2026-03-04T09:00:00-05:00
draft: false
tags: ["meta", "homelab"]
categories: ["general"]
summary: "Why this blog exists, and what's coming."
cover:
  image: ""
  alt: ""
  caption: ""
---

Every homelab needs a `hello-world` to prove the pipeline works end to end. This is mine.

## Why this blog

I run more infrastructure at home than is strictly reasonable, and I keep re-learning
the same lessons because I never wrote them down. So: a blog. Part **archive** (notes to
future me), part **showcase** (the work, in the open), part **thinking out loud**.

Expect posts on:

- **Homelab** — servers, storage, the physical and virtual plumbing that holds it together.
- **Networking** — DNS, VLANs, Cloudflare, the parts everyone pretends are simple.
- **Self-hosting & automation** — containers, CI, the small robots that do my chores.
- **Occasionally personal** — but never politics.

## How this site is built

Static site with [Hugo](https://gohugo.io/), [PaperMod](https://github.com/adityatelange/hugo-PaperMod)
theme, deployed to **GitHub Pages** via GitHub Actions, fronted by **Cloudflare**. The
canonical home is [blog.cst.nyc](https://blog.cst.nyc/); a few other domains 301-redirect
here so there's one source of truth.

```bash
# the whole publish loop
git commit -m "new post" && git push   # Actions builds + deploys the rest
```

More soon. If something here helps you, the comments are open.
