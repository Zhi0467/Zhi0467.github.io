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

- Site background should be `#FFFCE5`.
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

The short note for May 11, 2026 links to a static HTML file:

```markdown
[situation](/files/26-post3.html)
```

Keep standalone HTML notes under `files/` when linking them from posts.
