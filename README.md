# PageSpeed Optimizer

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code Skill](https://img.shields.io/badge/Claude%20Code-Skill-purple.svg)](https://docs.anthropic.com/en/docs/claude-code)
[![Version](https://img.shields.io/badge/version-2.0.0-green.svg)](CHANGELOG.md)

A Claude Code skill to achieve 100% on all 4 Google PageSpeed Insights categories.

```
┌─────────────────────────────────────────────────────────┐
│                   PageSpeed 100%                        │
├──────────────┬──────────────┬─────────────┬────────────┤
│ Performance  │Accessibility │Best Practices│    SEO    │
│  LCP < 2.5s  │    WCAG      │  Security   │   Meta    │
│  FCP < 1.8s  │    ARIA      │  Headers    │  Schema   │
│  CLS < 0.1   │  Contrast    │   HTTPS     │ Canonical │
│  TBT < 200ms │   Labels     │             │           │
└──────────────┴──────────────┴─────────────┴────────────┘
```

## Features

- **Framework Detection** - Automatically detects your project stack and applies relevant patterns
- **Complete Workflow** - Analyze, Fix, and Verify in a structured approach
- **Real Code Examples** - Copy-paste ready solutions for each framework
- **Prioritized Fixes** - Focus on high-impact optimizations first

## Supported Frameworks

| Framework | Status | Patterns File |
|-----------|--------|---------------|
| Laravel + Vite + Alpine.js | ✅ Ready | [laravel-patterns.md](laravel-patterns.md) |
| React + Vite | ✅ Ready | [react-patterns.md](react-patterns.md) |
| Next.js (App Router) | ✅ Ready | [nextjs-patterns.md](nextjs-patterns.md) |
| Vue 3 + Nuxt 3 | ✅ Ready | [vue-nuxt-patterns.md](vue-nuxt-patterns.md) |
| WordPress | 🚧 Coming Soon | [wordpress-patterns.md](wordpress-patterns.md) |

## Optimization Modules

| Module | Description | Guide |
|--------|-------------|-------|
| **Images** | WebP/AVIF conversion, srcset, compression | [images-optimization.md](images-optimization.md) |
| **Advanced** | Database queries, PWA, local caching | [advanced-optimization.md](advanced-optimization.md) |
| **Troubleshooting** | Common issues and solutions | [troubleshooting.md](troubleshooting.md) |

## Installation

### Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed

### Install the Skill

```bash
git clone https://github.com/anthropics/pagespeed-optimizer.git \
  ~/.claude/skills/pagespeed-optimizer
```

The skill is available immediately in all Claude Code sessions.

## Usage

### Slash Command

```bash
# Analyze a URL
/pagespeed-optimizer https://example.com

# Analyze an exported JSON report
/pagespeed-optimizer ./pagespeed-report.json

# Full project audit
/pagespeed-optimizer --audit

# Fix a specific category
/pagespeed-optimizer --fix performance
/pagespeed-optimizer --fix accessibility
```

### Natural Language

The skill activates automatically when you mention:

- "PageSpeed", "Core Web Vitals", "LCP", "FCP", "CLS"
- "optimize performance", "improve score"
- "WCAG accessibility", "security headers"

### Example Prompts

```
"My site has an LCP of 4.5s, help me optimize it"

"Here's my PageSpeed report, fix the accessibility issues"

"Add the missing security headers for Best Practices"

"Optimize this Laravel project for PageSpeed 100%"
```

## Workflow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   ANALYZE    │────▶│     FIX      │────▶│    VERIFY    │
├──────────────┤     ├──────────────┤     ├──────────────┤
│• Detect stack│     │• Performance │     │• Build prod  │
│• Read report │     │• Accessibility│    │• Re-test     │
│• Prioritize  │     │• Best Pract. │     │• Confirm 100%│
│              │     │• SEO         │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
```

### Priority System

| Priority | Impact | Examples |
|----------|--------|----------|
| CRITICAL | >10 pts | LCP image, render-blocking resources |
| HIGH | 5-10 pts | Unoptimized images, missing preconnect |
| MEDIUM | 2-5 pts | Cache headers, resource hints |
| LOW | <2 pts | Minor optimizations |

## Target Metrics

| Metric | Target | Category |
|--------|--------|----------|
| LCP (Largest Contentful Paint) | < 2.5s | Performance |
| FCP (First Contentful Paint) | < 1.8s | Performance |
| CLS (Cumulative Layout Shift) | < 0.1 | Performance |
| TBT (Total Blocking Time) | < 200ms | Performance |
| Contrast Ratio | 4.5:1 (text) | Accessibility |
| Security Headers | 6 headers | Best Practices |

## Real-World Example

See [EXAMPLES.md](EXAMPLES.md) for a complete case study showing how we achieved 100% on all categories for a real website.

**Quick summary:**
- Site: physio.chevassut.ch
- Before: Performance 98, Accessibility 94
- After: All categories at 100%
- Key fixes: Color contrast, touch target sizes

## File Structure

```
pagespeed-optimizer/
├── SKILL.md                    # Main skill definition
├── README.md                   # This file
├── LICENSE                     # MIT License
├── .gitignore
│
├── # Framework Patterns
├── laravel-patterns.md         # Laravel + Vite + Alpine.js
├── react-patterns.md           # React + Vite
├── nextjs-patterns.md          # Next.js (App Router)
├── vue-nuxt-patterns.md        # Vue 3 + Nuxt 3
├── wordpress-patterns.md       # WordPress (Coming Soon)
│
├── # Optimization Guides
├── images-optimization.md      # Image formats and tools
├── advanced-optimization.md    # Database, PWA, caching
├── troubleshooting.md          # Common issues
│
└── EXAMPLES.md                 # Real-world case studies
```

## Tools Used

The skill has access to these tools without requiring permission:

- `Read`, `Write`, `Edit` - File modification
- `Bash` - Build commands and diagnostics
- `Grep`, `Glob` - Code search
- `WebFetch` - Retrieve PageSpeed reports

## Running Lighthouse Locally

```bash
# Install Chrome if needed
npx @puppeteer/browsers install chrome@stable

# Run audit
CHROME_PATH=$(find $HOME/.cache/puppeteer -name chrome -type f 2>/dev/null | head -1) \
npx lighthouse https://example.com \
  --output=json \
  --output-path=./report.json \
  --chrome-flags="--headless --no-sandbox"
```

## Contributing

Contributions are welcome! Here's how you can help:

1. **Add a framework** - Create a new `*-patterns.md` file following the existing format
2. **Share a case study** - Add your success story to `EXAMPLES.md`
3. **Improve documentation** - Fix typos, add examples, clarify instructions
4. **Report issues** - Open an issue for bugs or feature requests

### How to Contribute

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-framework`)
3. Make your changes
4. Submit a pull request

## License

MIT - See [LICENSE](LICENSE) for details.

---

**Author:** Baptiste Chevassut
**Repository:** [github.com/anthropics/pagespeed-optimizer](https://github.com/anthropics/pagespeed-optimizer)
