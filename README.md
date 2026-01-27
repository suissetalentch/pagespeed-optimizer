# PageSpeed Optimizer

A Claude Code skill that helps achieve 100% Google PageSpeed Insights scores across Performance, Accessibility, Best Practices, and SEO.

## Overview

This skill provides a systematic methodology and framework-specific patterns for optimizing websites to achieve perfect PageSpeed scores. It works with various technology stacks and follows a prioritized approach based on impact.

## Features

- **Comprehensive audit methodology** - Step-by-step process from analysis to deployment
- **Core Web Vitals optimization** - LCP, FCP, CLS, TBT, and SI improvements
- **Framework-specific patterns** - Ready-to-use code for Laravel, React, and Next.js
- **Accessibility compliance** - WCAG-compliant form handling and navigation
- **Security headers** - Production-ready security configurations
- **Troubleshooting guide** - Solutions for common PageSpeed issues

## Installation

### For Claude Code Users

Copy this skill to your Claude Code skills directory:

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Clone this repository
git clone https://github.com/baptistmusic/pagespeed-optimizer.git ~/.claude/skills/pagespeed-optimizer
```

### Manual Installation

1. Download the contents of this repository
2. Place them in `~/.claude/skills/pagespeed-optimizer/`
3. Restart Claude Code to load the skill

## Usage

Once installed, you can invoke the skill in Claude Code:

```bash
# Analyze a PageSpeed report
claude "Use pagespeed-optimizer to analyze this report: [paste report or URL]"

# Get optimization suggestions
claude "Help me optimize my website performance using pagespeed-optimizer"

# Framework-specific help
claude "Show me Laravel patterns for PageSpeed optimization"
```

### Available Commands

| Command | Description |
|---------|-------------|
| `--audit` | Run a full site audit |
| `--fix-performance` | Focus on performance optimizations |
| `--fix-accessibility` | Focus on accessibility fixes |
| `[report-file]` | Analyze a specific PageSpeed report |
| `[url]` | Analyze a website URL |

## Optimization Areas

### Performance (Core Web Vitals)

- **LCP (Largest Contentful Paint)**: Hero image optimization, font loading, resource hints
- **FCP (First Contentful Paint)**: Critical CSS, render-blocking resource elimination
- **CLS (Cumulative Layout Shift)**: Image dimensions, font loading strategies
- **TBT (Total Blocking Time)**: Code splitting, lazy loading, bundle optimization

### Accessibility

- Form field labeling and autocomplete
- Button and link accessibility (aria-label, aria-hidden)
- Color contrast compliance
- Focus state management

### Best Practices

- Security headers (CSP, HSTS, X-Frame-Options)
- HTTPS enforcement
- Modern API usage

### SEO

- Meta tags optimization
- Structured data (JSON-LD)
- Crawlability improvements

## Framework Support

| Framework | Documentation |
|-----------|---------------|
| Laravel + Vite + Alpine.js | [laravel-patterns.md](laravel-patterns.md) |
| React + Vite | [react-patterns.md](react-patterns.md) |
| Next.js | [nextjs-patterns.md](nextjs-patterns.md) |

## Troubleshooting

Common issues and their solutions are documented in [troubleshooting.md](troubleshooting.md).

## File Structure

```
pagespeed-optimizer/
├── SKILL.md              # Main skill definition and methodology
├── laravel-patterns.md   # Laravel + Vite + Alpine.js patterns
├── react-patterns.md     # React + Vite patterns
├── nextjs-patterns.md    # Next.js patterns
├── troubleshooting.md    # Common issues and solutions
├── README.md             # This file
├── LICENSE               # MIT License
├── CONTRIBUTING.md       # Contribution guidelines
└── CHANGELOG.md          # Version history
```

## Contributing

Contributions are welcome! Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on how to submit pull requests.

### Ways to Contribute

- Add patterns for additional frameworks (Vue, Svelte, etc.)
- Improve existing optimization strategies
- Add troubleshooting scenarios
- Fix documentation errors

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Google PageSpeed Insights documentation
- Web.dev Core Web Vitals guidelines
- Claude Code team at Anthropic

## Author

**Baptiste Chevassut** - [baptiste@chevassut.ch](mailto:baptiste@chevassut.ch)

---

Made with Claude Code
