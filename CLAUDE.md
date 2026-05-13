# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start          # Dev server with live reload at http://localhost:8000
npm run build      # Build JS (ES5+ESM), CSS, and plugins to dist/
npm test           # Run ESLint + QUnit tests via Puppeteer
```

Individual gulp tasks:
```bash
npx gulp js        # Build JS bundles only (ES5 + ESM)
npx gulp css       # Build core CSS + all themes
npx gulp plugins   # Build plugin bundles
npx gulp eslint    # Lint js/** and gulpfile.js
npx gulp build -- --root /path/to/slides  # Build for a different root dir
npx gulp serve -- --port 8001             # Dev server on custom port
npm run build -- css-themes               # Build themes only
```

Tests run all `test/*.html` files via QUnit + Puppeteer (no way to run a single test file from CLI — open `http://localhost:8009/test/<file>.html` manually with the server running).

## Code Style

- Tabs for indentation (not spaces)
- Single-quoted strings
- ESLint config is in `package.json` (babel parser, ES6 browser env)

## Architecture

### Source vs. Distribution
- Source lives in `js/`, `css/`, `plugin/`
- Built output goes to `dist/` — **never edit `dist/` directly**
- Build tool: Gulp + Rollup + Babel + Sass

### Core JS (`js/`)
Entry point is [js/index.js](js/index.js), which exports both the modern class-based API (`new Reveal(element, options)`) and a backwards-compatible singleton API (`Reveal.initialize(options)`).

The main class is in [js/reveal.js](js/reveal.js). All feature areas are split into controller classes under [js/controllers/](js/controllers/):

| Controller | Responsibility |
|---|---|
| `autoanimate` | Auto-Animate transitions between slides |
| `backgrounds` | Slide background rendering (color, image, video, iframe) |
| `controls` | Navigation arrow UI |
| `fragments` | Step-through fragment animations |
| `keyboard` | Keyboard shortcut handling |
| `location` | URL hash / history management |
| `notes` | Speaker notes, postMessage to speaker window |
| `overview` | Overview (ESC) mode |
| `plugins` | Plugin registration and lifecycle |
| `printview` | PDF/print layout |
| `scrollview` | Scroll-mode layout |
| `slidecontent` | Lazy-loading media, iframes, data-src |
| `touch` | Touch/swipe input |

Utilities are in [js/utils/](js/utils/) (`util.js`, `device.js`, `color.js`, `constants.js`). Default config values are in [js/config.js](js/config.js).

### Build Outputs
- `dist/reveal.js` — UMD bundle (broad browser support via Babel+polyfills)
- `dist/reveal.esm.js` — ES module bundle (modern browsers only)
- `dist/theme/*.css` — Compiled themes (one per source `.scss` in `css/theme/source/`)

### Plugins (`plugin/`)
Six built-in plugins: `highlight`, `markdown`, `math`, `notes`, `search`, `zoom`. Each compiles to both `.js` (UMD) and `.esm.js` in its own directory. Plugins should **not** be submitted as PRs — they belong in separate repos.

### Themes (`css/theme/`)
Themes are written in Sass. Each theme in `css/theme/source/` follows the pattern:
1. Import `template/mixins.scss`
2. Import `template/settings.scss` (variable declarations)
3. Override variables / add custom styles
4. Import `template/theme.scss` (generates final CSS from variables)
