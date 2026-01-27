# Contributing to PageSpeed Optimizer

Thank you for considering contributing to PageSpeed Optimizer! This document provides guidelines and instructions for contributing.

## How to Contribute

### Reporting Issues

If you find a bug or have a suggestion:

1. Check existing issues to avoid duplicates
2. Open a new issue with a clear title and description
3. Include relevant details:
   - Framework you're using
   - PageSpeed report excerpt (if applicable)
   - Expected vs actual behavior

### Submitting Changes

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Make your changes
4. Commit with clear messages (`git commit -m 'Add Vue.js patterns'`)
5. Push to your branch (`git push origin feature/amazing-feature`)
6. Open a Pull Request

## Guidelines

### Adding Framework Patterns

When adding patterns for a new framework:

1. Create a new file: `[framework]-patterns.md`
2. Follow the existing structure:
   - Configuration examples
   - Font loading patterns
   - Image optimization
   - Script loading
   - Form accessibility
   - Security headers
3. Update `SKILL.md` to reference the new patterns
4. Update `README.md` framework support table

### Documentation Style

- Use clear, concise language
- Include code examples with comments
- Provide both "before" and "after" examples for fixes
- Use tables for quick reference
- Follow Markdown best practices

### Code Examples

Code examples should be:

- **Practical**: Ready to copy and use
- **Complete**: Include all necessary imports/context
- **Commented**: Explain non-obvious parts
- **Tested**: Verify the code works

Example structure:
```markdown
### Feature Name
```
Priority: HIGH/MEDIUM/LOW - Brief explanation
```

**Problem:** What PageSpeed reports

**Solution:**
```[language]
// Code example here
```

**Usage:**
```[language]
// How to use the solution
```
```

### Commit Messages

- Use present tense ("Add feature" not "Added feature")
- Use imperative mood ("Move cursor to..." not "Moves cursor to...")
- Limit first line to 72 characters
- Reference issues when applicable

Good examples:
- `Add Vue.js patterns for font optimization`
- `Fix accessibility example for icon buttons`
- `Update troubleshooting guide with CLS solutions`

## Development Setup

This skill is documentation-only, so there's no build process. To work on it locally:

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/pagespeed-optimizer.git

# Create a branch
git checkout -b my-contribution

# Make your changes and test with Claude Code
# Link to your local skills directory for testing
ln -s $(pwd) ~/.claude/skills/pagespeed-optimizer
```

## Code of Conduct

### Our Standards

- Be respectful and inclusive
- Accept constructive criticism gracefully
- Focus on what's best for the community
- Show empathy towards others

### Unacceptable Behavior

- Harassment or discriminatory language
- Personal attacks
- Publishing others' private information
- Other unprofessional conduct

## Questions?

Feel free to open an issue for any questions about contributing.

Thank you for helping improve PageSpeed Optimizer!
