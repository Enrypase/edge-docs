# GitHub Actions - Documentation Auto-Rebuild

This directory contains GitHub Actions workflows for automatically triggering website rebuilds when documentation changes.

## 📚 Documentation

- **[Setup Guide](./SETUP_GUIDE.md)** - Complete setup instructions with all required secrets
- **[Quick Reference](./QUICK_REFERENCE.md)** - Quick command reference and troubleshooting

## 🔄 Workflows

### `trigger-website-rebuild.yml`

Automatically triggers when:

- Changes are pushed to the `master` or `main` branch
- Files in the `content/` directory are modified
- Manual workflow dispatch

**What it does:**
Sends a `repository_dispatch` event to the `Enrypase/edge-test` repository with metadata about the commit that triggered it.

## 🚀 Quick Start

1. Follow the [Setup Guide](./SETUP_GUIDE.md) to configure secrets
2. Push changes to the `content/` directory
3. Watch the automatic rebuild at https://github.com/Enrypase/edge-test/actions

## 🔗 Related

- Website repository: https://github.com/Enrypase/edge-test
- Website rebuild workflow: `Enrypase/edge-test/.github/workflows/rebuild-on-docs-change.yml`
