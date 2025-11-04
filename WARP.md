# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

This is a Quartz v4 static site generator for publishing a digital garden/notes website. It's a TypeScript/Node.js project that transforms Markdown files into a static website.

- **Content source**: `content/` directory (Markdown files, images, Obsidian vault structure)
- **Output**: `public/` directory (static HTML/CSS/JS)
- **Package manager**: pnpm (v10.4.1+)
- **Node version**: 22+

## Common Commands

### Development

```bash
# Start local dev server with hot reload (default on port 8080)
pnpm dev
# or
npx quartz build --serve
```

### Building

```bash
# Build static site for production
pnpm build
# or
npx quartz build

# Build with verbose output
npx quartz build --verbose

# Build with bundle analysis
npx quartz build --bundleInfo
```

### Code Quality

```bash
# Type check and format check
pnpm check

# Format code
pnpm format

# Run tests
pnpm test
```

### Quartz CLI Commands

```bash
# Create/initialize Quartz setup
npx quartz create

# Update to latest Quartz version
npx quartz update

# Restore content from cache
npx quartz restore

# Sync with Git (commit, pull, push workflow)
npx quartz sync
```

## Architecture

### Core Build Pipeline

The build system follows a three-stage plugin architecture:

1. **Transformers** (`quartz/plugins/transformers/`): Parse and transform Markdown content
   - Parse frontmatter, process LaTeX, handle syntax highlighting
   - Transform links (Obsidian, GitHub Flavored Markdown)
   - Generate metadata (descriptions, table of contents, last modified dates)

2. **Filters** (`quartz/plugins/filters/`): Filter which content gets published
   - Remove drafts, apply explicit publish rules

3. **Emitters** (`quartz/plugins/emitters/`): Generate output files
   - Content pages, folder pages, tag pages
   - Assets, static files, 404 page
   - Search index, RSS feed, sitemap, CNAME

### Configuration Files

- **`quartz.config.ts`**: Main configuration (plugins, theme, analytics, i18n settings)
- **`quartz.layout.ts`**: Page layout configuration (components for header, footer, sidebar, etc.)
- **`tsconfig.json`**: TypeScript configuration (uses Preact for JSX)

### Key Directories

- **`quartz/components/`**: Preact/TSX UI components (Graph, Explorer, Search, Darkmode, etc.)
- **`quartz/plugins/`**: Plugin system (transformers, filters, emitters)
- **`quartz/cli/`**: CLI implementation (build, create, update, sync commands)
- **`quartz/util/`**: Utilities (path handling, performance timing, dep graph, etc.)
- **`quartz/styles/`**: SCSS stylesheets
- **`quartz/static/`**: Static assets copied to output

### Build System Details

- Uses **esbuild** for bundling TypeScript/SCSS
- **Chokidar** for file watching in dev mode
- **WorkerPool** for parallel processing
- Dependency graph tracking for fast rebuilds (`--fastRebuild`)
- WebSocket-based hot reload in dev mode

### Content Processing

1. Markdown files parsed with **unified/remark/rehype** pipeline
2. Supports Obsidian-flavored Markdown (wikilinks, embeds)
3. Math rendering via KaTeX or MathJax
4. Syntax highlighting via Shiki
5. Graph visualization via D3.js
6. Search powered by FlexSearch

## Project-Specific Notes

### Content Structure

- Content lives in `content/` directory (not version controlled output)
- Ignore patterns: `private`, `templates`, `.obsidian` (see `quartz.config.ts`)
- Chinese locale (`zh-CN`) with Noto Sans SC font

### Deployment

- GitHub Actions CI on PRs and pushes to `v4` branch
- Docker build workflow creates container images on tags
- Uses GitHub Container Registry (ghcr.io)

### Custom Configuration

- **Giscus comments** enabled (repo: zh30/zhanghe.date)
- **Plausible analytics** configured
- **Social image generation** enabled
- Theme uses custom color scheme (light/dark mode)

## Development Workflow

1. Content authors edit Markdown files in `content/`
2. Run `pnpm dev` to preview changes locally
3. Quartz watches for changes and rebuilds incrementally
4. Build output goes to `public/` (gitignored)
5. Run `pnpm check` before committing code changes
6. Use `npx quartz sync` for content synchronization workflow

## Testing

Tests use `tsx` to run TypeScript directly:

- Path utilities: `quartz/util/path.test.ts`
- Dependency graph: `quartz/depgraph.test.ts`

Run with `pnpm test` or individually: `tsx ./quartz/util/path.test.ts`

## Plugin Development

To add custom functionality:

1. Create transformer/filter/emitter in respective `quartz/plugins/` subdirectory
2. Export from `quartz/plugins/index.ts`
3. Add to plugin array in `quartz.config.ts`
4. Follow existing plugin patterns (see `quartz/plugins/types.ts` for interfaces)

## Component Development

Components are Preact/TSX files in `quartz/components/`:

- Export a function that returns a `QuartzComponent`
- Use `QuartzComponentProps` for type safety
- Register in `quartz.layout.ts` for different page types
- Components can be positioned: `beforeBody`, `left`, `right`, `afterBody`, `header`, `footer`
