# AGENTS.md

Plugin collection for opencode v2. Each plugin is a zero-dependency,
self-contained JS file under `plugins/`, designed to be dropped into
`~/.config/opencode/plugins/` (global) or `.opencode/plugins/` (project).

## Conventions

- One file per plugin, engine inlined — no shared imports between plugins,
  no npm deps beyond `@opencode-ai/plugin` (already resolves from
  `~/.config/opencode/node_modules` at opencode runtime).
- Keep plugins provider-agnostic: no references to LM Studio, local server
  names, or machine-specific hosts. Config lives in a `CONFIG` block at the
  top of each file; blank URL fields mean "feature off".
- `plugins/web-search.js` engine derives from `web-search-plugin` v1.0.1 by
  thrilok (MIT) — keep the attribution line in its header. Original source:
  `~/opt/web-search` (git@github.com:amitpandey-io/web-search.git),
  `opencode/web-search.js`.

## Smoke test

`@opencode-ai/plugin` is not in this repo. ESM resolves bare imports relative
to the *importing file*, so the plugin must be imported from the test dir
(which holds the symlinked `node_modules`), not from the repo:

```sh
mkdir -p /tmp/oc-test/node_modules && cd /tmp/oc-test
ln -sf ~/.config/opencode/node_modules/@opencode-ai node_modules/@opencode-ai
cp <path>/web-search.js .
node --input-type=module -e '
import { WebSearch } from "file:///tmp/oc-test/web-search.js";
const def = (await WebSearch()).tool.web_search;
const r = await def.execute({ query: "opencode plugins" }, { abort: new AbortController().signal });
console.log(r.output);
'
```

## Opencode v2 plugin facts (from opencode.ai/docs/plugins, Oct 2026)

- Files in `~/.config/opencode/plugins/` or `.opencode/plugins/` auto-load at
  startup; npm plugins go in the `plugin` array of opencode.json instead.
- A plugin tool with the same name as a built-in tool overrides the built-in.
- Built-in `websearch` is gated (OpenCode provider or `OPENCODE_ENABLE_EXA` /
  `OPENCODE_ENABLE_PARALLEL`); built-in `webfetch` is keyless and unrestricted.
