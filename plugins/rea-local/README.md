# REA Local

A Windows local plugin that connects an MCP-capable desktop or Codex host to
[REA](https://github.com/morluto/rea)'s existing server. REA is maintained by
morluto; this plugin integration is maintained in Ananay28425/rea.

## What is included

- A portable Agent Plugins 1.0 manifest and stdio MCP configuration.
- A Codex compatibility manifest and MCP configuration.
- The repository's existing `reverse-engineer-anything` skill and its reference
  guides, copied without modification from commit
  `6389e50337c50f285c0979d9182b2489ea78e32b`.
- The upstream MIT license with its original copyright notice.

The server is the published upstream npm package `rea-agents@6.7.0`, pinned in
both MCP configuration files. It is not a source build of this fork. The skill
comes from the supplied fork; repository main can describe newer behavior than
the published package. Always use the connected server's actual tools and input
schemas as the authority.

The first launch uses npm's cache and may download the pinned package and its
dependencies. It does not run `rea setup`, rewrite other clients' configuration,
or install analysis engines.

## Requirements

- Windows with `cmd.exe`, npm, and a supported Node.js runtime on the host's PATH.
  REA requires Node 22.x >=22.19, Node 24.x >=24.11, or Node 26+.
- A host that supports local stdio MCP servers and local plugins.
- The local files or applications selected for investigation.
- Existing tools for the requested analysis. Static JavaScript and .NET analysis
  do not require Hopper or Ghidra. Native decompilation needs an applicable
  provider; Windows Ghidra support is experimental. See
  [REA installation](https://github.com/morluto/rea/blob/main/docs/installation.md).

A normal remote ChatGPT chat cannot access a laptop's filesystem through this
package. Account import alone does not start the local MCP process. This is not
a hosted MCP endpoint.

## Install from this repository

Check out the branch containing this package, then open a terminal at the
repository root:

```powershell
codex plugin marketplace add .
```

This registers the repository marketplace at
`.agents/plugins/marketplace.json`. In a compatible ChatGPT desktop app, open
Plugins, choose **Ananay's REA Plugins**, and install **REA Local**. Restart or
start a new session as the host requires. Use your installed CLI's
`codex plugin --help` if its authoring commands differ.

The marketplace is a local source. The manifest uses
`cmd.exe /d /c npx -y rea-agents@6.7.0 mcp`, following REA's documented Windows
stdio registration. The Codex compatibility configuration gives startup 60
seconds; a first package download may need a longer host timeout.

Avoid installing a second independent REA MCP registration through
`rea setup` when using this plugin. If REA is already registered, select one
connection in the host so the skill and tools have an unambiguous server.

## Verify the connection

1. Confirm a **rea** MCP server and the **reverse-engineer-anything** skill
   appear in a new session.
2. Ask the host to list REA's currently advertised tools. A skill appearing does
   not establish that its MCP server connected.
3. Use a small JavaScript application directory that you own as a static
   analysis target. Check that `analyze_javascript_application` returns evidence
   for the supplied path. Static results do not establish runtime behavior.
4. Verify a native provider separately only when your task requires one.
   Neither npm package availability nor tool discovery proves provider readiness.

This package was prepared through the GitHub API in a session without a local
execution environment. Manifest consistency and source references were checked;
npm startup, live MCP discovery, actual analysis, and Windows provider behavior
remain unverified.

## Use

Example prompts after connection:

> Use REA to inspect this local Electron application and trace how its search
> feature works. Cite the evidence and identify unresolved questions.

> Compare these two local application versions and explain which observed
> changes affect the feature.

Use the supplied skill's summary and evidence-paging workflows. Preserve
observations, derivations, inferences, unknowns, coverage limits, and evidence
provenance. Runtime observation uses the permissions and prerequisites of the
local host.

## Export a plugin archive

From the repository root, use PowerShell to include compatibility files as well
as normal files. The temporary staging folder is separate from the source.
Choose an output ZIP path that does not already exist.

```powershell
$stage = Join-Path ([IO.Path]::GetTempPath()) ("rea-plugin-" + [guid]::NewGuid())
New-Item -ItemType Directory -Path $stage | Out-Null
Copy-Item -LiteralPath .\\plugins\\rea-local -Destination $stage -Recurse -Force
tar -a -c -f .\\rea-local-0.1.0.zip -C $stage rea-local
Remove-Item -LiteralPath $stage -Recurse -Force
```

Inspect the archive before importing it. It should contain one `rea-local/`
directory, both manifests, both MCP configuration files, the complete skill and
references, README, and LICENSE. Do not include the repository marketplace in
the archive. Do not add npm caches, `node_modules`, credentials, binaries, or
analysis targets.

## Other host operating systems

The checked-in MCP commands target Windows. On Linux or macOS, change the
`rea` entry in both `mcp.json` and `.mcp.json` to the following command and
arguments, keeping each file's existing schema/wrapper and any host timeout:

```json
{
  "command": "npx",
  "args": ["-y", "rea-agents@6.7.0", "mcp"]
}
```

REA's supported operations and required providers vary by host. Check the
upstream guides rather than inferring support from the presence of a tool name.
