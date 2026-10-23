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

## Install

### Option A — ask opencode (any OS, one paste)

Paste this into an opencode session — opencode resolves its own config
directory and shell, so this works on macOS, Linux and Windows:

```txt
Install an opencode plugin:
1. Find your global opencode plugin directory (the directory containing your global opencode.json, plus a "plugins" subfolder; usually ~/.config/opencode/plugins). Create it if missing.
2. Download https://raw.githubusercontent.com/amitpandey-ai/opencode-plugins/main/plugins/web-search.js into that directory as web-search.js
3. Confirm the file was written and tell me to restart opencode to load it.
```

### Option B — manual

Global plugin directory:

| OS | Path |
|---|---|
| macOS / Linux | `~/.config/opencode/plugins/` |
| Windows (PowerShell) | `$HOME\.config\opencode\plugins\` (i.e. `C:\Users\<you>\.config\opencode\plugins\`) |

> If `XDG_CONFIG_HOME` is set, use `$XDG_CONFIG_HOME/opencode/plugins/`.

macOS / Linux:

```sh
mkdir -p ~/.config/opencode/plugins
curl -fsSL -o ~/.config/opencode/plugins/web-search.js \
  https://raw.githubusercontent.com/amitpandey-ai/opencode-plugins/main/plugins/web-search.js
```

Windows (PowerShell):

```powershell
New-Item -ItemType Directory -Force "$HOME\.config\opencode\plugins" | Out-Null
Invoke-WebRequest -Uri https://raw.githubusercontent.com/amitpandey-ai/opencode-plugins/main/plugins/web-search.js `
  -OutFile "$HOME\.config\opencode\plugins\web-search.js"
```

Project-level instead of global (same on every OS): put the file in
`.opencode/plugins/` inside your project — it loads the same way.

No `opencode.json` changes are needed — files in the plugins directories are
auto-loaded at startup. The `"plugin": [...]` array in `opencode.json` is only
for plugins published as **npm packages**.

After installing, **restart opencode** — plugins load at startup.

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
