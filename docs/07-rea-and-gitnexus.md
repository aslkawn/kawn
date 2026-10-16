# REA and GitNexus: how they help

## REA (morluto/rea, npm `rea-agents` 6.5.0, MIT)
Reverse-engineering MCP/CLI for binaries, apps, runtime behaviour. For a JS web app the relevant command is
`rea analyze-javascript-application <path> --json`. Value for Kawn: structured map of GEV's modules, data flows,
network calls -> fills thin spots in specs 03/04.
Safety: do NOT run `npx rea-agents setup` (edits Claude config). Run from the local clone after reading
package.json scripts and source. Output goes to `/Users/imdadareeph/Desktop/42/wallA-notes/` (outside `kawn/`).

## GitNexus (local, v1.6.8)
Code knowledge graph, exposed to Claude as MCP `gitnexus` (user scope). Indexed: gods-eye-view (31,302 symbols),
rea (46,831). Web UI http://localhost:4280 with `&skipGraph=0` for big graphs. Use for: "what calls X", feature-to-module
maps, cluster lists. Restart Claude Code / `/mcp` to see its tools.

## Workflow (Wall A -> Wall B)
1. Query GitNexus/REA on gods-eye-view for one thin area (e.g. weather, director).
2. Write behaviour-only notes in `wallA-notes/` (no identifiers, no code).
3. Leak-check, rewrite as spec text, copy into `kawn/specs/`.
4. Builder session in `kawn/` never sees the original.
