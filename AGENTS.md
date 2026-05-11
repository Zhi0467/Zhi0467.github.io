# Repository Instructions

This is a Jekyll 3 / AcademicPages personal website.

## Source Layout

- Edit source files, not `_site/`. `_site/` is generated output.
- Main site config is `_config.yml`.
- Pages live in `_pages/`; blog posts live in `_posts/`.
- Sass lives in `_sass/` and is imported by `assets/css/main.scss`.
- Blog archive rendering is split between `_pages/year-archive.html` and `_includes/archive-single.html`.
- Blog thumbnails come from post front matter under `header.teaser`; teaser paths are relative to `/images/`.

## Current Design State

- Site background should be `#FFFCE5`.
- Link styling is defined in `_sass/_links.scss`, imported late from `assets/css/main.scss`.
- Text links should be monospace, underlined, and dark text color, not blue.
- Blog archive thumbnail sizing is in `_sass/_archive.scss`.
  - Desktop blog thumbnails are currently `200px`.
  - Mobile blog thumbnails are currently `108px`.
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
```

The preview command above uses `--no-watch`, so restart the server after editing Sass, layouts, includes, posts, or pages.

## Validation Commands

Check Sass compilation:

```sh
bundle exec ruby -e 'require "sass"; source = File.read("assets/css/main.scss").sub(/\A---\s*\n---\s*\n/m, ""); Sass::Engine.new(source, syntax: :scss, load_paths: ["_sass"], cache: false).render; puts "sass ok"'
```

Check Liquid parsing for the modified blog archive files:

```sh
bundle exec ruby -e 'require "liquid"; ["_includes/archive-single.html", "_pages/year-archive.html"].each { |path| Liquid::Template.parse(File.read(path)); puts "liquid ok #{path}" }'
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

This keeps local preview pages from loading stale assets from `https://Zhi0467.github.io/...`.

## Post Notes

The short note for May 11, 2026 links to a static HTML file:

```markdown
[situation](/files/26-post3.html)
```

Keep standalone HTML notes under `files/` when linking them from posts.
