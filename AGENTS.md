# AGENTS.md

Plugin collection for opencode v2. Each plugin is a zero-dependency,
self-contained JS file under `plugins/`, designed to be dropped into the
`plugins/` folder of the active opencode config dir (check with
`opencode debug paths`; usually `~/.config/opencode/plugins/`) or
`.opencode/plugins/` (project).

## Conventions

- One file per plugin, engine inlined — no shared imports between plugins,
  no npm deps (v2.0.32 registers tools with plain JSON Schema; the v1
  `tool()` helper from `@opencode-ai/plugin` is obsolete here).
- Keep plugins provider-agnostic: no references to LM Studio, local server
  names, or machine-specific hosts. Config lives in a `CONFIG` block at the
  top of each file; blank URL fields mean "feature off".
- `plugins/web-search.js` engine derives from `web-search-plugin` v1.0.1 by
  thrilok (MIT) — keep the attribution line in its header. Original source:
  `~/opt/web-search` (git@github.com:amitpandey-io/web-search.git),
  `opencode/web-search.js`.

## Smoke test

The plugin is a plain ESM file with no imports — import it anywhere and
drive the contract directly:

```sh
node --input-type=module -e '
const { default: plugin } = await import("file:///path/to/web-search.js");
console.log("id:", plugin.id);
let def;
plugin.setup({ tool: { transform: (cb) => { cb({ add: (t) => { def = t; } }); return Promise.resolve({ dispose(){} }); } } });
const r = await def.execute({ query: "opencode plugins" }, { signal: AbortSignal.timeout(20000) });
console.log(r.content);
'
```

## Opencode v2 plugin facts (from opencode.ai/docs/plugins, Oct 2026)

- Files in `~/.config/opencode/plugins/` or `.opencode/plugins/` auto-load at
  startup; npm plugins go in the `plugin` array of opencode.json instead.
  File plugins do NOT need an opencode.json entry (confirmed in v2.0.32 logs
  + `packages/core/src/plugin/source-directory.ts`). Plugin load failures are
  logged to `~/.local/share/opencode/log/opencode.log` ("failed to load plugin").
- Paths are XDG-based on every OS (verified in `packages/util/src/global-roots.ts`):
  global config dir is `$XDG_CONFIG_HOME/opencode` or `~/.config/opencode` —
  on native Windows that is `C:\Users\<name>\.config\opencode` (`~` = user profile).
- A plugin tool with the same name as a built-in tool overrides the built-in.
- Built-in `websearch` is gated (OpenCode provider or `OPENCODE_ENABLE_EXA` /
  `OPENCODE_ENABLE_PARALLEL`); built-in `webfetch` is keyless and unrestricted.
- File plugins are discovered via the module's **default export**, which must
  be `{ id, setup(context) }` (or `{ id, effect }` for Effect-native). A named
  export alone is silently skipped; a default without `id`+`setup`/`effect`
  fails with `Plugin must export a default definition...` in
  `~/.local/share/opencode/log/opencode.log`.
- Tools register inside `setup` via `context.tool.transform(editor =>
  editor.add({ name, description, input: <JSON Schema>, execute }))`;
  `execute(input, ctx)` returns `{ content }` (v1 `{tool: {...}}` hooks and
  `tool()` definitions are dead code in 2.0.32 — no error, just no tools).
- Plugins **hot-reload**: adding/changing a file in the config dir's
  `plugins/` activates it in a running server (no restart needed). Verify with
  `GET <server>/api/plugin` (Basic auth `opencode:<password>`; service
  password in `<config dir>/service.json`) — look for the id with
  `state: active`.
- `context.websearch.transform` providers are validated against
  `WebSearch.Result` (`packages/schema/src/websearch.ts`): `time` is a
  **required** struct (`{ published?: number }`) even though `title`/`content`
  are optional. Returning results without `time` fails the host-side decode →
  `503 {"_tag":"ServiceUnavailableError","message":"Web search request failed: <id>"}`.
  Always emit `time: {}`. Verified on v2.1.4: file plugins DO receive
  `context.websearch` in `setup`; provider `execute` gets
  `({ query }, { signal })`; live test `POST /api/websearch`.
- `effect` for schema-decode tests resolves from the opencode config dir's
  `node_modules`: `createRequire("/Users/apandey/opencode-config/x.js")("effect")`.
