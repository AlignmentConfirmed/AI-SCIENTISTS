# Upgrade: Documentation Format

## Priority: 2 (Low urgency, high quality-of-life)
## Complexity: LOW — tooling exists
## Status: READY WHEN NEEDED

---

## The Problem

Markdown (.md) is the correct format for GitHub repositories.
However, as the documentation grows (currently 27+ files), the
reading experience degrades — no search, no sidebar navigation,
no cross-document linking beyond manual hyperlinks.

## The Current State

Markdown works. GitHub renders it. Scientists can read it. No
action needed for initial release.

## The Upgrade Path

When the documentation exceeds ~50 files or when the code is
added:

**Option A — mdBook (Rust ecosystem)**
- Takes existing .md files, zero rewriting
- Generates a searchable, navigable website
- Sidebar with chapter structure
- Built-in search
- Deploys to GitHub Pages for free

**Option B — Docusaurus (JavaScript ecosystem)**
- Same concept, more customization
- Better for mixed code + documentation
- Larger community

**Option C — Jupyter Book (Python ecosystem)**
- Best for code + documentation + visualization
- Native support for executable notebooks
- Ideal when the science engine has Python bindings

## When To Upgrade

- After initial public release and feedback
- When the file count exceeds comfortable GitHub browsing
- When code examples need to be executable alongside docs
- When a contributor offers to do it

## Estimated Effort

One afternoon for mdBook. One day for Docusaurus. Two days for
Jupyter Book. The existing .md files are the input. No rewriting.

---

*Not urgent. The current format works. Upgrade when the content
outgrows the container.*
