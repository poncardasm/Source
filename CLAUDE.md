# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Source is Ghost's current default theme, built with Handlebars templating and a component-based architecture. This is a standalone theme that can be developed and deployed independently.

**Repository**: https://github.com/TryGhost/Source
**Demo**: https://source.ghost.io
**Ghost Version**: >=5.0.0
**Node.js Version**: 22.21.1 (specified in `.nvmrc`)

## Development Commands

All commands should be run from the theme directory (`/Users/mchael/gh-repo/ghost-local/content/themes/source`):

```bash
# Install dependencies
yarn install

# Development mode with livereload (watches CSS, JS, and HBS files)
yarn dev

# Build assets only (compiles CSS and JS to assets/built/)
gulp build

# Create distribution zip for theme upload
yarn zip

# Run theme validation with gscan
yarn test

# Run validation in CI mode (fails on errors, verbose output)
yarn test:ci
```

**Note**: `yarn dev` is the primary development command - it runs `gulp` which builds assets, starts livereload server, and watches for file changes.

## Architecture

### Template Hierarchy

**Main templates** (root level):
- `default.hbs` - Parent template with global structure, loaded by all other templates
- `home.hbs` - Homepage template
- `index.hbs` - Main post list/archive template
- `post.hbs` - Individual post template
- `page.hbs` - Individual page template
- `tag.hbs` - Tag archive template
- `author.hbs` - Author archive template

**Custom templates by slug**:
- `page-{slug}.hbs` - Custom page template (e.g., `page-about.hbs` for `/about/`)
- `tag-{slug}.hbs` - Custom tag template (e.g., `tag-news.hbs` for `/tag/news/`)
- `author-{slug}.hbs` - Custom author template (e.g., `author-ali.hbs` for `/author/ali/`)

### Partials Structure

**Components** (`partials/components/`):
- `header.hbs` - Site header wrapper
- `header-content.hbs` - Header content variations (Landing, Highlight, Magazine, Search)
- `navigation.hbs` - Site navigation
- `footer.hbs` - Site footer
- `cta.hbs` - Call-to-action/signup component
- `featured.hbs` - Featured posts carousel
- `post-list.hbs` - Post list/grid rendering

**Shared partials** (`partials/`):
- `post-card.hbs` - Individual post card component
- `feature-image.hbs` - Responsive feature image rendering
- `email-subscription.hbs` - Email signup form
- `lightbox.hbs` - Image lightbox functionality
- `search-toggle.hbs` - Search toggle button

**Icons** (`partials/icons/`):
- Inline SVG icons included via `{{> "icons/icon-name"}}`
- Add custom SVG icons by creating new `.hbs` files in this directory

**Typography** (`partials/typography/`):
- `fonts.hbs` - Font preloading and definitions for sans, serif, and mono options

### Asset Pipeline (Gulp + PostCSS)

**Source files**:
- `assets/css/screen.css` - Main CSS entry point (imports other CSS files)
- `assets/js/lib/*.js` - Third-party library files (loaded first)
- `assets/js/*.js` - Custom JavaScript files

**Build output** (`assets/built/`):
- `screen.css` + `screen.css.map` - Compiled and minified CSS
- `source.js` + `source.js.map` - Concatenated and uglified JavaScript

**Build process** (configured in `gulpfile.js`):
1. CSS: PostCSS with easy-import → autoprefixer → cssnano
2. JavaScript: Concatenate lib files + custom files → uglify
3. Sourcemaps generated for both CSS and JS
4. Livereload triggers on file changes during `yarn dev`

**Do not edit `assets/built/` directly** - these files are auto-generated and will be overwritten.

## Theme Configuration

The theme exposes extensive customization through Ghost admin (Design > Theme settings), defined in `package.json` under `config.custom`:

**Layout & Style**:
- `navigation_layout`: Logo in middle/left/stacked
- `site_background_color`: Custom background color
- `header_and_footer_color`: Background or Accent color
- `title_font`: Modern sans-serif, Elegant serif, or Consistent mono
- `body_font`: Modern sans-serif or Elegant serif

**Homepage Settings** (`group: homepage`):
- `header_style`: Landing, Highlight, Magazine, Search, or Off
- `header_text`: Custom header text (defaults to site description)
- `background_image`: Toggle publication cover as background
- `show_featured_posts`: Display featured posts carousel
- `post_feed_style`: List or Grid layout
- `show_images_in_feed`: Toggle post images in list view
- `show_author`, `show_publish_date`, `show_publication_info_sidebar`: Metadata toggles

**Post Settings** (`group: post`):
- `show_post_metadata`: Display post metadata
- `enable_drop_caps_on_posts`: Enable drop caps on first paragraph
- `show_related_articles`: Display related articles

**Access custom settings in templates**: `{{@custom.setting_name}}`

## Ghost Templating (Handlebars)

**Common helpers**:
- `{{asset "path"}}` - Reference theme assets with cache-busting
- `{{ghost_head}}` - Required: outputs SEO meta, structured data, Ghost settings
- `{{body_class}}` - Dynamic body classes based on context
- `{{@site.*}}` - Site settings (title, description, locale, etc.)
- `{{@custom.*}}` - Custom theme settings
- `{{img_url image size="m"}}` - Responsive image URLs
- `{{> "partial-name"}}` - Include partial template

**Image sizes** (defined in `package.json`):
- xs: 160px, s: 320px, m: 600px, l: 960px, xl: 1200px, xxl: 2000px

**Full theme API**: https://ghost.org/docs/themes/

## Color Contrast Calculation

The theme automatically calculates text color (dark/light) based on the custom background color. This logic is in `default.hbs` lines 27-44, using YIQ color space algorithm. The class `has-light-text` or `has-dark-text` is applied to the `<html>` element for CSS styling.

## Theme Validation

Ghost requires themes to pass validation before upload:

```bash
yarn test        # Standard validation
yarn test:ci     # CI mode with verbose output and fatal errors
```

**Validation tool**: `gscan` (https://github.com/TryGhost/gscan)
**Rules validated**: Theme structure, required templates, Ghost API usage, deprecated features

**Before deploying**: Always run `yarn test` - the `yarn zip` command runs `yarn pretest` (which runs `gulp build`) automatically.

## Distribution & Release

**Create theme zip**:
```bash
yarn zip  # Creates dist/source.zip
```

The zip excludes: `node_modules/`, `dist/`, `yarn.lock`, `yarn-error.log`, `gulpfile.js`

**Release process** (maintainers only):
```bash
yarn ship  # Checks git status, versions, tags, and pushes
```

Post-ship, `gulp release` creates a GitHub release draft with changelog.

## File Watching & Livereload

When running `yarn dev`, Gulp watches:
- `assets/css/**` → rebuilds CSS
- `assets/js/**` → rebuilds JavaScript
- `*.hbs` and `partials/**/*.hbs` → triggers livereload

Livereload port is managed by `gulp-livereload` - browser auto-refreshes on changes.

## Integration with Ghost

This theme directory is part of a local Ghost installation at `/Users/mchael/gh-repo/ghost-local/`.

**Ghost commands** (run from Ghost root):
```bash
ghost start  # Start Ghost on http://localhost:2368
ghost stop   # Stop Ghost
```

**Admin interface**: http://localhost:2368/ghost/

Changes to theme files take effect immediately when Ghost is running in development mode. For production, themes must be uploaded as zip files through the admin interface.
