---
title: LLM-Friendly Docs
weight: 5
prev: /docs/tools/mcp/
next: /docs/internals/
---

This documentation is published in machine-readable formats so that large language models and AI agents can consume it directly, without scraping rendered HTML.

## llms.txt

A single index of the whole documentation is available at [/llms.txt](https://docs.nxthdr.dev/llms.txt), following the [llms.txt convention](https://llmstxt.org/). It lists every page with a short summary and a direct link, giving an agent a compact map of the docs in one request.

## Per-page Markdown

Every page is also available as raw Markdown. Append `.md` to a page's path, or use `index.md` for a section's landing page. For example:

- [/docs/peering/getting-started.md](https://docs.nxthdr.dev/docs/peering/getting-started.md)
- [/docs/probing/probes.md](https://docs.nxthdr.dev/docs/probing/probes.md)
- [/docs/peering/index.md](https://docs.nxthdr.dev/docs/peering/index.md) (section landing page)

This gives agents clean, structure-preserving content to ingest instead of HTML.

## Querying the data

To go beyond the documentation and query the open datasets in natural language, see the [MCP Servers](/docs/tools/mcp/) page.
