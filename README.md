# jupyter-mcp

JupyterLab CRDT MCP server extension layer for OpenCharly images.

The `jupyter-mcp` candy pip-installs [FastMCP](https://gofastmcp.com/) and the
in-tree `jupyter_mcp` Python package into the parent pixi environment, then
writes a `jupyter_server_config.d` drop-in that enables `jupyter_mcp` as a
Jupyter Server extension. It is a **Tier 1 post-install candy** with no
`pixi.toml` of its own — it installs into whatever pixi environment the parent
`jupyter` / `jupyter-ml` layer provides.

The extension registers a Streamable HTTP MCP server at
`http://localhost:8888/mcp` exposing notebook tools (`notebook_*`, `cell_*`,
`room_list`, `notebook_list_users`). Cell operations mutate the live CRDT
document, so MCP clients and JupyterLab browser users edit the same notebook
simultaneously.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `jupyter-mcp` |
| Depends on | a parent Tier 2 pixi env (`jupyter` / `jupyter-ml`) |
| Python packages | `fastmcp>=3.2.0`, in-tree `jupyter_mcp` |
| MCP endpoint | `http://localhost:8888/mcp` (Streamable HTTP) |
| Tools | `notebook_list`, `notebook_create`, `notebook_get`, `notebook_watch`, `notebook_list_users`, `cell_get`, `cell_update`, `cell_insert`, `cell_delete`, `cell_execute`, `room_list` |
| Install files | `charly.yml`, `jupyter_mcp/` (Python package) |
| Service / port | none (extension of the parent Jupyter server) |

## How to use it

Compose the layer by pinning this repo in a box's `candy:` list alongside a
Tier 2 Jupyter parent (the `jupyter` candy lives in `opencharly/pod-jupyter`):

```yaml
my-jupyter:
  candy:
    base: fedora
    candy:
      - '@github.com/opencharly/pod-jupyter:v2026.243.0410'
      - '@github.com/opencharly/layer-jupyter-mcp:v2026.239.1633'
```

After the Jupyter box is deployed, the server routes `/mcp`:

```bash
# an empty POST returns 400 (handler present), not 404
curl -s -o /dev/null -w '%{http_code}\n' -X POST http://localhost:8888/mcp
```

## Layout

- `charly.yml` — the `jupyter-mcp:` candy entity: the pip `run:` steps, the
  extension-enable step, the `check:` assertions, and the embedded `skill:`
  entity.
- `jupyter_mcp/` — the in-tree Python package (`app.py`, `mcp_server.py`,
  `rtc_adapter.py`, `tornado_asgi.py`, `pyproject.toml`).
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-jupyter:jupyter-mcp` — the CRDT MCP server extension
- Parent candies: `/charly-jupyter:jupyter` (lightweight), `/charly-jupyter:jupyter-ml` (GPU)
- Downstream MCP consumers: `/charly-hermes:hermes`, `/charly-openwebui:openwebui`
- Sibling MCP provider: `/charly-selkies:chrome-devtools-mcp`
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella
