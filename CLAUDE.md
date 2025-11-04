# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start

This is a Quartz v4 installation - a static site generator for building digital gardens from Markdown files.

### Essential Commands

```bash
# Development - build and serve with live reload
pnpm run dev
# or
npx quartz build --serve

# Production build
pnpm run build
# or
npx quartz build

# Type check and linting
pnpm run check

# Format code
pnpm run format

# Run tests
pnpm run test

# Initialize new Quartz site
npx quartz create

# Update Quartz to latest version
npx quartz update

# Sync content with git
npx quartz sync
```

## Code Architecture

### Build Pipeline

Quartz processes content through a three-stage pipeline:

1. **Parse** (`quartz/processors/parse.ts`)
   - Converts Markdown files to AST (Abstract Syntax Tree)
   - Uses `remark` for Markdown parsing and `rehype` for HTML conversion
   - Supports parallel processing via worker threads for large sites
   - Each file goes through: Text → Markdown AST → HTML AST

2. **Filter** (`quartz/processors/filter.ts`)
   - Filters processed content based on plugin rules
   - Example: Remove drafts, unpublished content

3. **Emit** (`quartz/processors/emit.ts`)
   - Generates final output files (HTML, CSS, assets)
   - Each emitter plugin can output multiple files
   - Supports both array and async generator return types

### Plugin System

Plugins are organized into three categories (defined in `quartz/plugins/types.ts`):

- **Transformers** (`quartz/plugins/transformers/`): Modify content during parsing
  - Examples: `FrontMatter.ts`, `SyntaxHighlighting.ts`, `Latex.ts`, `GitHubFlavoredMarkdown.ts`
  - Implement `markdownPlugins()` and `htmlPlugins()` to hook into processing

- **Filters** (`quartz/plugins/filters/`): Remove content from processing
  - Example: `RemoveDrafts.ts` filters out drafts

- **Emitters** (`quartz/plugins/emitters/`): Generate output files
  - Examples: `ContentPage.ts`, `TagPage.ts`, `Assets.ts`
  - Each emitter returns files to write to disk
  - Can implement `partialEmit()` for incremental rebuilds in watch mode

### Configuration

Main configuration in `quartz.config.ts`:

- **Global configuration**: Page title, theme, SPA settings, analytics, locales
- **Plugins**: Lists of transformers, filters, and emitters to use

Key config properties:

- `configuration.pageTitle`: Site title
- `configuration.theme`: Colors, fonts, typography
- `configuration.enableSPA`: Single-page app routing
- `configuration.baseUrl`: Used for absolute URLs in feeds/sitemaps

### Component System

UI components in `quartz/components/`:

- Built with **Preact** (React-like framework)
- Component types defined in `quartz/components/types.ts`
- Styling with **SCSS** in component subdirectories
- Layout components in `quartz/layout.ts`

Common components:

- `PageTitle.tsx`, `Breadcrumbs.tsx`, `TagList.tsx`
- `Search.tsx`, `Graph.tsx`, `Backlinks.tsx`
- `Explorer.tsx` (file tree navigation)

### CLI and Build System

CLI handlers in `quartz/cli/handlers.js`:

- JavaScript implementation (not TypeScript)
- Uses `@clack/prompts` for interactive CLI
- Build compilation via **esbuild**

Build workflow:

1. Transpiles TypeScript with esbuild to `.quartz-cache/`
2. Runs build pipeline from `quartz/build.ts`
3. Watches source files for changes (in `--serve` mode)

### File Structure

```
quartz/
├── components/          # Preact UI components
├── plugins/            # Plugin system
│   ├── emitters/       # Output generators
│   ├── filters/        # Content filters
│   └── transformers/   # Content transformers
├── processors/         # Build pipeline stages
├── util/              # Utilities (path, log, theme, etc.)
├── cli/               # CLI handlers (JS)
├── cfg.ts             # Configuration types
└── build.ts           # Main build orchestration

content/               # User content (Markdown files)
docs/                 # Quartz documentation
quartz.config.ts      # Main configuration
```

### Key Utilities

- **Path handling** (`quartz/util/path.ts`): Slugification, file path conversion
- **Performance tracking** (`quartz/util/perf.ts`): Build timing
- **Logging** (`quartz/util/log.ts`): Structured logging
- **Theme system** (`quartz/util/theme.ts`): Dynamic theming support

## Development Notes

### Creating Plugins

See `docs/advanced/making plugins.md` for detailed guide.

A plugin is a JavaScript object with:

```typescript
{
  name: string
  markdownPlugins?: (ctx: BuildCtx) => Plugin[]
  htmlPlugins?: (ctx: BuildCtx) => Plugin[]
  textTransform?: (ctx: BuildCtx, text: string) => string
  emit?: (ctx: BuildCtx, content, resources) => FilePath[] | AsyncGenerator
  partialEmit?: (ctx: BuildCtx, content, resources, changes) => FilePath[] | AsyncGenerator
}
```

### Creating Components

See `docs/advanced/creating components.md` for detailed guide.

Components are Preact functional components:

```typescript
type QuartzComponent = (props: ComponentProps) => ComponentReturn
```

### Watch Mode

When running with `--serve` or `--watch`:

- Uses `chokidar` to watch for file changes
- Incremental rebuilds only process changed files
- Emitters can implement `partialEmit()` for efficient updates
- WebSocket connection triggers client-side refresh

### Concurrency

Parsing uses worker threads for large sites:

- Single-threaded for < 128 files
- Multi-threaded (up to 4 workers) for larger sites
- Chunk size: 128 files per worker

## Site-Specific Configuration

This site is configured as:

- **Title**: "张赫的日常"
- **Language**: zh-CN (Simplified Chinese)
- **Base URL**: zhanghe.date
- **Font**: Noto Sans SC (Chinese) for body, IBM Plex Mono for code
- **Plugins enabled**: FrontMatter, OFM, GFM, Syntax highlighting, LaTeX, TOC, Citations, RSS, etc.

Content structure:

- Markdown files in `content/`
- Ignore patterns: `["private", "templates", ".obsidian"]`
- Date type: "modified" (use file modification date)
