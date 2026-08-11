# Comments (giscus)

Comments run on **[giscus](https://giscus.app)**, which stores each thread as a
**GitHub Discussion** in this repo. Readers sign in with GitHub to comment.

The wiring is already in place (`layouts/partials/comments.html` +
`[params.giscus]` in `hugo.toml`). It stays **dormant** until you fill in two IDs,
so the site builds fine before then. To turn it on:

1. **Make the repo public** (giscus can't read Discussions on a private repo for anonymous readers).
2. Repo → **Settings → General → Features** → tick **Discussions**.
3. Install the giscus app: <https://github.com/apps/giscus> → **Configure** →
   grant it access to **`charlestephen/blog`**.
4. Go to <https://giscus.app>, and under **Configuration**:
   - Repository: `charlestephen/blog`
   - Page ↔ Discussions mapping: **pathname**
   - Discussion category: **Announcements** (or make a "Comments" category first;
     pick one that only maintainers can post new threads in)
5. giscus generates a snippet — copy the two values:
   - `data-repo-id`  → paste into `hugo.toml` → `[params.giscus] repoId`
   - `data-category-id` → paste into `hugo.toml` → `[params.giscus] categoryId`
6. Commit + push. Comments now appear at the bottom of every post
   (disabled on the About page via its front matter).

To disable comments on a single post, add `comments: false` to that post's front matter.
