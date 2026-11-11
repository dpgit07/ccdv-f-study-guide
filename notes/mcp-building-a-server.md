# MCP — Building and Connecting a Server — Notes

**Course used:** [Claude Certified Developer Foundations (CCDV-F) Exam Prep](https://www.udemy.com/course/claude-certified-developer-foundations-exam-prep/) by Kuljot Singh Bakshi (Udemy). Lab code: the instructor's public [lab repo](https://github.com/kuljotSB/Claude-Certified-Developer-CCDV-F); the code below is my own adaptation of standard SDK patterns shown in the labs, with generic example names. Credit to the instructor.
**Lab files (in the course's lab repo):** `MCP-Servers/` (`server/server.py`, `Dockerfile`, `requirements.txt`, `MCP.ipynb`)
**Exam weight:** feeds "Tools and MCPs" (~10.6% unofficial) and official objective 5 (MCP without leaking credentials).
Companion to [section-06-tool-use.md](section-06-tool-use.md) §8 (the *client* side of MCP) and Section 8 (vaults).

> *(general)* = not shown in the lab code.

---

## 1. What an MCP server is
A **standard way to expose tools** so any MCP-capable client (the Claude API connector, the Agent SDK, Managed Agents)
can discover and call them **without custom glue per integration**. Write the server once; many clients use it.

**DE analogy:** a **JDBC/ODBC driver** or a service with a published API contract: one standard protocol, any client.

---

## 2. The server (`server.py`): a few lines of Python
```python
from mcp.server.fastmcp import FastMCP
mcp = FastMCP(name="catalog-server", host="0.0.0.0", port=8000)

@mcp.tool(name="get_item_by_rating",
          description="Return catalog items rated at or above the given minimum.",
          structured_output=True)
def get_item_by_rating(min_rating: float) -> Dict[str, Any]:
    ...
    return {"items": matched_items}

mcp.run(transport="streamable-http", mount_path="/mcp")
```
- **`@mcp.tool(...)` turns a typed Python function into an MCP tool.** The input schema comes from the **type hints**
  (same idea as `@beta_tool` in Section 6).
- **`description` is what Claude reads to decide when to call it**, the same rule as every tool so far. The lab defines
  a handful of clear, distinct lookup tools (by name, by id, by rating, and so on).
- **`structured_output=True`** → the tool returns structured data (a dict) rather than free text, which links back to
  Section 4 (structured outputs): predictable, machine-readable results.
- **Not-found handling:** returns `{"error": "Item not found"}`, a structured, explicit error Claude can act on. This
  is the right instinct (no silent failure), though MCP also has its own error flag *(general)*.
- **Transport `streamable-http`** = a **remote** server reachable by URL. (The other common style is a local process
  over stdio *(general)*.)

## 3. Deploying it (`Dockerfile`)
- Python 3.11-slim image → `pip install -r requirements.txt` → copy code → `EXPOSE 8000` → `CMD ["python", "server.py"]`.
- **Why a container:** the API's MCP connector needs to reach the server **by a URL**. In the Messages API flow it's Anthropic's
  platform connecting to your server, so `localhost` won't do; it needs a reachable address *(general)*.

## 4. Connecting Claude to it (`MCP.ipynb`)
```python
mcp_servers=[{"type": "url", "url": mcp_server_url, "name": "catalog_server"}],
tools=[{"type": "mcp_toolset", "mcp_server_name": "catalog_server"}],
betas=["mcp-client-2025-11-20"]
```
- Same shape as Section 6: **`mcp_servers` (where) + `mcp_toolset` (which server's tools)**; the server URL comes
  from an env var (`MCP_SERVER_URL`), not hard-coded.
- A prompt such as "which catalog items are rated above 4, and which has the most reviews?" needs **no loop**: the platform
  calls the MCP tools itself, so Claude picks `get_item_by_rating` on its own. (Loop rule: any *client-side* tool means you own a loop.)

## 5. Three ways MCP shows up in the course: keep them apart
| Where | Who connects to the MCP server | Notebook |
|---|---|---|
| Messages API MCP connector | Anthropic's API, on your request | `Tool-Use/MCP.ipynb`, `MCP-Servers/MCP.ipynb` |
| Agent SDK | Your SDK process (`mcp_servers={...}`, tools named `mcp__<server>__<tool>`) | `Agents-SDK/Multi-Tool.py` |
| Managed Agents | The hosted agent, with secrets from a **vault** | `Managed-Agents/MCP-Connector.ipynb`, `Vaults.ipynb` |

## 6. Judgment points
- **This server has no authentication.** Fine for read-only demo data; for anything sensitive or with write tools you need authN/authZ, and
  you'd pass credentials via a **vault** (Section 8), never in prompts.
- **Keep the tool list focused.** Six well-described tools is good; dozens → consider tool search (Section 10).
- **Descriptions and names are the interface.** Overlapping tools (for example a general `get_items` vs `get_item_by_rating`) are distinguished only by their descriptions.
- Beyond tools, MCP also defines **resources and prompts** *(general; the lab only uses tools)*.

## 7. Likely exam angles *(my reading)*
1. **What problem MCP solves:** one standard interface instead of bespoke integrations per client.
2. **Remote server needs a reachable URL**; credentials via vault/secret store.
3. **Good tool descriptions and typed parameters** drive correct tool use.
4. **Connector vs SDK vs Managed Agents:** who makes the connection and who holds the secrets.

## 8. Self-check
1. How does `@mcp.tool` know the tool's input schema?
2. Why does `MCP.ipynb` have no `while` loop?
3. Why does a server called by the Messages API connector need a URL (not `localhost`)?
4. A server exposes a `delete_record` tool with no auth. What's wrong?

*Answers: (1) the function's type hints. (2) the platform resolves MCP tool calls itself, and no client-side tools are declared. (3) Anthropic's platform makes the connection, so it must be reachable. (4) anyone with the URL can call it; add authentication/authorization, scope the token, and require approval for destructive actions.*
