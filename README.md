# Security blog — Hugo + Void theme

Static blog. Hugo builds it, GitHub Actions deploys it to GitHub Pages.

## Local development

```bash
hugo server -D      # -D also renders drafts -> http://localhost:1313
```

Requirements (already installed on this machine):

- Hugo **extended** 0.169.0 — `winget install -e --id Hugo.Hugo.Extended`
- Node.js + `npm ci` — the theme compiles its CSS with `css.TailwindCSS`,
  which shells out to the Tailwind CLI in `node_modules`. Hugo alone is not enough.

## Writing a post

```bash
hugo new content posts/http-request-smuggling-basics.md
```

Uses `archetypes/posts.md`, so the front matter skeleton and section headings are
pre-filled. Set `draft = false` when it is ready. See the draft post
`content/posts/writing-cheatsheet.md` for every markup feature the theme supports
(callouts, code blocks, KaTeX, series, tables, footnotes).

## Deploying

Remote: `git@github.com:yadro29a/yadro29a.github.io.git` → https://yadro29a.github.io/

1. Repo **Settings → Pages → Source → GitHub Actions** (one-time, in the web UI).
2. `git push origin main`. `.github/workflows/hugo.yml` builds with Hugo extended,
   runs `npx pagefind --site public` to generate the search index, and deploys.

Search only works on the deployed site — the Pagefind index is built after Hugo,
so `hugo server` shows a "not available in this build" notice on `/search/`.

## Theme overrides in `layouts/`

The theme lives in `themes/void` (git submodule, pinned at commit `3db9a58`).
Never edit it directly — files in `layouts/` shadow the theme's copies. Four are
overridden on purpose:

| File | Why |
|---|---|
| `partials/head.html` | Removed the Busuanzi view counter (third-party script from `busuanzi.ibruce.info` loaded on every page). Also calls `head/css.html` uncached: Hugo ≥ 0.169 refuses `templates.Defer` inside `partialCached`, which breaks the theme's production build. |
| `partials/footer.html` | Removed the site-wide Busuanzi counter markup. |
| `_default/single.html` | Removed the per-post Busuanzi view count. |
| `partials/head/css.html` | Dropped the Noto Serif SC (CJK) stylesheet — English-only blog, saves ~60 KB gzipped and one request. |

Result: the built site loads **zero third-party resources**. Fonts, KaTeX, and
search are all self-hosted. Verify after a build with:

```bash
grep -rhoE '(src|href)="https?://[^"]+' public --include='*.html'   # expect no output
```

When you update the theme (`git submodule update --remote themes/void`),
re-check these four files against the theme's new versions.

## Notes

- `security.exec.allow` in `hugo.toml` must list `tailwindcss` or the CSS build
  fails with "not whitelisted in policy".
- `markup.highlight.noClasses = false` is required for dark/light code themes.
- Two deprecation warnings (`.Language.LanguageCode`, `.Language.LanguageDirection`)
  come from the theme's `baseof.html`. Harmless; they will need a theme update,
  or an override of `baseof.html`, before they become errors.
