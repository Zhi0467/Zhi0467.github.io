# Repository Instructions

This is a Jekyll 3 / AcademicPages personal website.

The current public site shape is intentionally small:

- `/` Home/About
- `/blog/` Blog archive
- `/research/` Research
- `/sitemap/` A curated public sitemap linked from the footer

Legacy AcademicPages sample sections such as publications, talks, teaching, portfolio, CV page archives, category/tag archive pages, and markdown generator notebooks have been removed. Do not reintroduce those collection pages unless the user explicitly asks.

## Source Layout

- Edit source files, not `_site/`. `_site/` is generated output.
- Main site config is `_config.yml`.
- Pages live in `_pages/`; blog posts live in `_posts/`.
- Sass lives in `_sass/` and is imported by `assets/css/main.scss`.
- Blog archive rendering is split between `_pages/year-archive.html` and `_includes/archive-single.html`.
- Blog thumbnails come from post front matter under `header.teaser`; teaser paths are relative to `/images/`.
- Standalone public files live under `files/`; currently keep `files/CV.pdf` and `files/26-post3.html`.
- Current image assets are limited to favicon/app icons, `images/me.jpg`, and images used by posts/social cards. Avoid adding back unused theme sample images.
- `AGENTS.md` is excluded from Jekyll output in `_config.yml`; keep it as repo-only maintenance documentation.

## Current Design State

The site ships two skins, chosen by a visitor-facing style toggle:

- `html.style-retro` (default): a System-7 "paper" look ported from a standalone
  note. Paper background `#f3f0e7` with an 8px grid, the whole page wrapped in a
  bordered window with a pinstriped title bar, Chicago/Geneva headings, Monaco
  links and code, 2px black borders with hard offset shadows.
- `html.style-classic`: the original AcademicPages look, background `#FFFCE5`.

Skin rules:

- All retro styling lives in `_sass/_retro.scss`, imported last from
  `assets/css/main.scss` (after `_dark-mode.scss`). Everything except a short
  "chrome is inert" block at the top is inside `@mixin retro-skin`, applied to
  `html.style-retro`; the dark variant is `@mixin retro-dark`.
- `_links.scss` and `_dark-mode.scss` style links through
  `a:not(#goog-wm-sb):not(.btn)`. The id inside `:not()` gives those rules
  id-level specificity, so every retro link rule must carry the same qualifier.
  `_retro.scss` keeps it in the `$a` variable; use `#{$a}` instead of bare `a`.
- The window markup (`.desktop` > `.window` > `.titlebar`) is in
  `_layouts/default.html` and is neutralized (title bar hidden, no border, no
  max-width) unless the retro skin is on.
- Post-level components (`.hero`, `.lede`, `.post-toc`, `.callout`, `.note`,
  `.compare`, `.card`, `.mathbox`, `.timeline`, `.tiny`, `.sources`,
  `.post-footer-rule`) must be styled for BOTH skins: the classic versions live
  in `_sass/_post-components.scss`, the retro overrides in `_retro.scss` under
  `.page__content`. Adding a component to only one file leaves the other skin
  rendering raw markup.
  Use `.post-toc`, not `.toc`: `.toc` is the theme's own uppercase TOC widget.
- The theme pins `.sidebar` to `position: fixed` above 1024px, which only lines
  up with the classic fixed masthead. The retro skin resets it to `static`;
  otherwise the author profile drifts over the body text and the footer.
- MathJax output is scaled by `chtml: { scale: ... }` in `_includes/head/custom.html`
  (currently `0.88`); it applies to every post.
- Both toggles sit in `.masthead__controls` in `_includes/masthead.html`, styled
  by `_sass/_masthead.scss` (classic) and `_retro.scss` (retro squares). The
  style toggle's icon is a CSS half-filled square, not a Font Awesome glyph;
  the bundled Font Awesome subset does not include `fa-palette`.
- The style choice persists in `localStorage["style"]` (`retro` | `classic`),
  applied early in `_includes/head.html` and wired up in `_includes/scripts.html`,
  same pattern as the dark mode toggle.
- Dark mode is controlled by the icon button at the right side of the top bar.
  - The early theme script is in `_includes/head.html`.
  - The click/persistence script is in `_includes/scripts.html`.
  - Dark styles live in `_sass/_dark-mode.scss`, imported late from `assets/css/main.scss`.
- Top navigation is centered in `_sass/_navigation.scss`.
  - The active/current nav item is assigned in `_includes/masthead.html` and styled by `.masthead__menu-item--active` in `_sass/_masthead.scss`.
  - Blog posts should mark `Blog` active.
- Link styling is defined in `_sass/_links.scss`, imported late from `assets/css/main.scss`.
- Text links should be monospace, underlined, and dark text color, not blue.
- Browser `<title>` should use the current page title, not constant `Home`; this is handled in `_includes/seo.html`.
- Social cards use existing post thumbnails, not separate images.
  - `_includes/seo.html` resolves social image metadata from `header.image`, `header.overlay_image`, then `header.teaser`.
  - X/Twitter card image URLs must be absolute and lowercase under `https://zhi0467.github.io`.
- Blog archive thumbnail sizing is in `_sass/_archive.scss`.
  - Desktop blog thumbnails are currently `200px`.
  - Mobile blog thumbnails are currently `108px`.
- Blog pages do not show the author sidebar.
  - Post defaults set `author_profile: false`.
  - `_pages/year-archive.html` sets `author_profile: false`.
  - `_layouts/archive.html` and `_layouts/single.html` add `main--no-sidebar`; blog archive/posts also add `main--blog`.
  - `.main--blog` currently has `max-width: 960px` on desktop to create wider side margins.
- `/sitemap/` should remain curated by hand in `_pages/sitemap.md`; do not loop over `site.pages`.
- Blog post comments use the theme's `custom` provider with an Utterances widget in `_includes/comments-providers/custom.html`.
- Utterances requires the GitHub app to be installed/authorized for `Zhi0467/Zhi0467.github.io` before visitors can create comments.
- Facebook has been removed from `_includes/social-share.html`; share buttons currently include Twitter/X and LinkedIn.

## Local Preview

The site uses `jekyll-github-metadata`, which may try to reach GitHub. For local preview, use this offline-safe command:

```sh
bundle exec ruby -e 'require "set"; require "jekyll"; require "jekyll-github-metadata"; require "jekyll-github-metadata/client"; module Jekyll; module GitHubMetadata; class Client; def internet_connected?; false; end; end; end; end; ARGV.replace(["serve", "--host", "127.0.0.1", "--port", "4000", "--no-watch", "--trace"]); load Gem.bin_path("jekyll", "jekyll")'
```

Preview URLs:

```text
http://127.0.0.1:4000/
http://127.0.0.1:4000/blog/
http://127.0.0.1:4000/sitemap/
```

The preview command above uses `--no-watch`, so restart the server after editing Sass, layouts, includes, posts, or pages.

If the shell has no UTF-8 locale (`LANG` empty), Sass conversion fails with
`Invalid US-ASCII character`. Prefix preview/build/validation commands with
`LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8`.
If port `4000` is already in use, use another local port such as `4001`.

## Validation Commands

Check Sass compilation:

```sh
bundle exec ruby -e 'require "sass"; source = File.read("assets/css/main.scss").sub(/\A---\s*\n---\s*\n/m, ""); Sass::Engine.new(source, syntax: :scss, load_paths: ["_sass"], cache: false).render; puts "sass ok"'
```

Check Liquid parsing for common modified layout/include/page files:

```sh
bundle exec ruby -e 'require "liquid"; ["_includes/archive-single.html", "_includes/head.html", "_includes/masthead.html", "_includes/scripts.html", "_layouts/archive.html", "_layouts/single.html", "_pages/year-archive.html", "_pages/sitemap.md"].each { |path| Liquid::Template.parse(File.read(path)); puts "liquid ok #{path}" }'
```

Offline build command:

```sh
bundle exec ruby -e 'require "set"; require "jekyll"; require "jekyll-github-metadata"; require "jekyll-github-metadata/client"; module Jekyll; module GitHubMetadata; class Client; def internet_connected?; false; end; end; end; end; ARGV.replace(["build", "--trace"]); load Gem.bin_path("jekyll", "jekyll")'
```

Jekyll builds may remove tracked `.jekyll-metadata`. If that happens and the user did not ask to change it, restore it:

```sh
git restore .jekyll-metadata
```

## Local URL Gotcha

`_includes/base_path` is intentionally root-relative:

```liquid
{% assign base_path = site.baseurl | default: "" %}
```

This keeps local preview pages from loading stale assets from `https://zhi0467.github.io/...`.

## Post Notes

`_posts/2026-09-22-single-rollout-agent-rl.html` is an HTML post, not Markdown,
because its body is hand-written HTML with heavy `\(...\)` / `\[...\]` MathJax.
Keep long-form notes that use the retro post components in `.html` so kramdown
does not touch the escapes.

The short note for May 11, 2026 links to a static HTML file:

```markdown
[situation](/files/26-post3.html)
```

Keep standalone HTML notes under `files/` when linking them from posts.
