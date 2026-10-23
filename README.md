# opencode-plugins

Zero-dependency, single-file [opencode](https://opencode.ai) v2 plugins.
Each plugin is one `.js` file with no npm dependencies beyond
`@opencode-ai/plugin` (already available at opencode runtime) — no build
step, just drop it into your plugins directory.

## Plugins

| Plugin | Tool | What it does |
|---|---|---|
| [`web-search`](plugins/web-search.js) | `web_search` | Keyless tiered web search: SearXNG (if configured) → DuckDuckGo HTML → Bing HTML. Returns title/URL/snippet per hit. Optional local-embeddings rerank. |

opencode's built-in `websearch` tool only works with the OpenCode provider
(or `OPENCODE_ENABLE_EXA`/`OPENCODE_ENABLE_PARALLEL`). `web_search` works
with **any** provider — including local models — without any API key.

## Install (simple prompt)

Paste this into an opencode session:

```txt
Install the web-search plugin for opencode:
1. mkdir -p ~/.config/opencode/plugins
2. curl -fsSL -o ~/.config/opencode/plugins/web-search.js https://raw.githubusercontent.com/amitpandey-ai/opencode-plugins/main/plugins/web-search.js
3. Tell me when done (opencode must be restarted for the plugin to load).
```

Then restart opencode.

Or do it manually:

```sh
mkdir -p ~/.config/opencode/plugins
curl -fsSL -o ~/.config/opencode/plugins/web-search.js \
  https://raw.githubusercontent.com/amitpandey-ai/opencode-plugins/main/plugins/web-search.js
```

Project-level instead of global? Put the file in `.opencode/plugins/` inside
your project — it loads the same way.

## Usage

The `web_search` tool takes:

| Arg | Type | Default |
|---|---|---|
| `query` | string (required) | — |
| `maxResults` | int 3–20 | `8` |

Example prompt: *"Search the web for the latest Bun release notes and summarize
them."*

## Configure

Config is a fixed `CONFIG` block at the top of the plugin file — edit after
copying, then restart opencode:

| Key | Default | Meaning |
|---|---|---|
| `searxngUrl` | `""` | Base URL of a self-hosted SearXNG instance (e.g. `http://localhost:8080`). Blank = use built-in DDG/Bing. |
| `embeddingsUrl` | `""` | OpenAI-compatible embeddings base URL for reranking results (e.g. a local server). Blank = rerank off. |
| `rerankModel` | `nomic-embed-text` | Embedding model used for rerank. |
| `locale` | `en-us` | Language/region for results. |
| `time` | `y` | Recency: `d`, `w`, `m`, `y`, or `""` (any). |
| `defaultMax` | `8` | Results per query (3–20). |

## License / attribution

The search engine derives from `web-search-plugin` v1.0.1 by thrilok
([LM Studio web-search plugin](https://lmstudio.ai/altra/web-search)),
MIT licensed; the attribution line in the plugin header is kept intact.
