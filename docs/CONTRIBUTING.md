---
  title: Contributing
  layout: default
  nav_order: 1
---

To add a new feature comparison, [submit a PR](https://github.com/igor-makarov/IsSwiftLikeRust/compare) by adding a new file to the `features/` directory using the following markdown template:

```markdown
---
title: <title>
layout: feature
feature-status:
  swift:
    status: (pending|supported|unavailable)
    details-caption: [optional, e.g. SE-####]
    details: <url>
  rust:
    status: (pending|supported|unavailable)
    details-caption: [optional, e.g. RFC]
    details: <url>
excerpt: >- 
  Short feature description, a couple of sentences.
  This will appear on the main page and on the feature page.

---

## Swift
Swift full description, verbose.
## Rust
Rust full description, verbose.

```