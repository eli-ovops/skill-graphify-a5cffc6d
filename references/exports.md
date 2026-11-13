# graphify reference: extra exports and benchmark

Load this when the user passed one of the export flags (`--wiki`, `--neo4j`, `--neo4j-push`, `--falkordb`, `--falkordb-push`, `--svg`, `--graphml`, `--mcp`), or when the corpus is large enough for the token-reduction benchmark. Each step runs only for its own flag.

File exports, database pushes, service startup, and host configuration are distinct actions. Execute only the ones covered by current specific authorization. A push flag does not override a higher-priority credential or account restriction. Never request, read, or export passwords/keys merely to follow this reference; if an allowed authentication mechanism cannot satisfy the governing rules, produce the permitted local export and report the blocked push. Do not infer an endpoint from a CLI default when the intended object is unknown. Examples are capability documentation, not instructions to start a server or modify configuration.

### Step 6b - Wiki (only if --wiki flag)

**Only run this step if `--wiki` was explicitly given in the original command.**

Run this before Step 9 (cleanup) so `.graphify_labels.json` is still available.

```bash
graphify export wiki
```

### Step 7 - Neo4j export (only if --neo4j or --neo4j-push flag)

**If `--neo4j`** - generate a Cypher file for manual import:

```bash
graphify export neo4j
```

**If `--neo4j-push`** - push directly to Neo4j using the destination URI supplied with the original `/graphify` command. Only push when this flag was explicitly given.

Pass the exact URI from the original `/graphify` command as one argument. When constructing a shell command, quote the URI for the shell that will execute the command (use `shlex.quote` for POSIX shells), or use a structured argument interface. Keep the URI as data; never interpolate it unquoted or evaluate it as shell code. Never replace a supplied URI with localhost. If the intended endpoint is missing, resolve that material target before pushing; CLI defaults alone are not authorization. Do not print the URI or credentials in status output or generated artifacts.

```bash
graphify export neo4j --push 'bolt://remote-host:7687' --user neo4j
```

The CLI documents `bolt://localhost:7687` and user `neo4j` as defaults; these are not proof of the user's intended target. `NEO4J_PASSWORD` is an implementation option for separately authorized controlled authentication, not permission for the agent to inspect the environment or obtain a password. If credential loading is forbidden, do not push. MERGE can reduce duplicate identities but does not make every write harmless or authorize blind retries; verify an uncertain result before retrying.

### Step 7a - FalkorDB export (only if --falkordb or --falkordb-push flag)

**If `--falkordb`** - generate a Cypher file. The statements are OpenCypher, but FalkorDB's `GRAPH.QUERY` runs one statement at a time (no bulk script import like Neo4j's `cypher-shell`), so prefer `--falkordb-push` to load a graph. Use this only when you want the portable `cypher.txt` artifact:

```bash
graphify export falkordb
```

**If `--falkordb-push`** - push directly to FalkorDB using the destination URI supplied with the original `/graphify` command. Only push when this flag was explicitly given.

Pass the confirmed destination as one structured argument or correctly shell-quoted data (`shlex.quote` for POSIX). Never evaluate it as shell code, replace it with localhost, or print a secret-bearing URI. Resolve a material unknown target before pushing. `FALKORDB_PASSWORD` documents the CLI's controlled-authentication option; do not ask for or inspect a password under this Skill. If current higher-priority rules prohibit the required credential use, stop only the push and retain a local export.

```bash
graphify export falkordb --push 'falkordb://remote-host:6379'
```

The CLI defaults are `falkordb://localhost:6379` and graph `graphify` (`redis://` or `host:port` also work). Confirm these match the authorized object; do not infer authority from defaults or optional authentication. MERGE reduces duplicates but does not prove every repeat is safe; inspect uncertain results before another write.

### Step 7b - SVG export (only if --svg flag)

```bash
graphify export svg
```

### Step 7c - GraphML export (only if --graphml flag)

```bash
graphify export graphml
```

### Step 7d - MCP server (only if --mcp flag)

Start only when the current task specifically authorizes this service, object, purpose, and lifetime. Do not register it in a desktop/global configuration merely because an example follows; that is a separate configuration change.

```bash
$(cat graphify-out/.graphify_python) -m graphify.serve graphify-out/graph.json
```

This starts a stdio MCP server that exposes tools: `query_graph`, `get_node`, `get_neighbors`, `get_community`, `god_nodes`, `graph_stats`, `shortest_path`. Add to Claude Desktop or any MCP-compatible agent orchestrator so other agents can query the graph live.

To configure in Claude Desktop, add to `claude_desktop_config.json`. Claude Desktop can't run `$(...)`, and under `uv tool install` the system `python3` can't import graphify — so set `command` to the **absolute interpreter path** printed by `cat graphify-out/.graphify_python`:
```json
{
  "mcpServers": {
    "graphify": {
      "command": "<absolute path from: cat graphify-out/.graphify_python>",
      "args": ["-m", "graphify.serve", "/absolute/path/to/graphify-out/graph.json"]
    }
  }
}
```

### Step 8 - Token reduction benchmark (only if total_words > 5000)

When the user requests a benchmark or its result would materially help the current task within budget, the following command is available. Corpus size alone does not mandate an extra benchmark:

```bash
graphify benchmark
```

Print the output directly in chat. If `total_words <= 5000`, skip silently - the graph value is structural clarity, not token compression, for small corpora.
