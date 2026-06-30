# ARCNET : Specification for Knowledge Graph

This document specifies a **file format and folder convention** for a knowledge graph stored as plain Markdown. It is **tool-agnostic**: it depends on no program, library, or language. Any producer (a person in an editor, or an extraction program) and any consumer (Obsidian, Juggl, a static-site generator, `grep`, a custom indexer) interoperate through this format.

A reference example conforming to this specification lives in [`examples/`](examples/) and simulates a graph built from six documents.

## Documents

- [`ARCNET-CORE.md`](ARCNET-CORE.md) — the domain-agnostic core: node model, identity, edges, citations, core kinds, merges, version control, and patch format.
- [`ARCNET-AST.md`](ARCNET-AST.md) — runtime-agnostic in-memory model: a plain-JSON representation of the graph as nodes of text, links, and headers, a lossless round-trip of the Markdown files.
- [`ARCNET-DOMAIN-ARTICLE.md`](ARCNET-DOMAIN-ARTICLE.md) — reference domain profile for ingesting research articles (`hypothesis`, `aporia`).
- [`ARCNET-CORE-THOUGHT.md`](ARCNET-CORE-THOUGHT.md) — extension adding the `thought` node kind for Core Thoughts distilled from a source's propositions, domain-agnostic.


## License

[![See LICENSE](https://img.shields.io/github/license/fogfish/arcnet-spec.svg?style=for-the-badge)](LICENSE)

