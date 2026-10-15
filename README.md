# ArcMap MCP Server

An MCP server that connects an MCP-compatible client to a running ArcMap installation through a local ArcMap Add-In. It provides tools for working with ArcMap projects, layers, map views, geoprocessing, and layout elements.

## Requirements

- Windows with ArcGIS Desktop / ArcMap 10.x installed
- Python 3.10 or newer for this MCP server
- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- An MCP-compatible client, such as Visual Studio Code with MCP support
- The companion ArcMap MCP Add-In installed and running in ArcMap

The ArcMap Add-In is not included in this repository. Install it separately, open ArcMap, and use its **Start MCP Server** toolbar button before connecting an MCP client. The server connects to `127.0.0.1:27173` by default.

## Installation

1. Clone this repository and install its dependencies:

   ```powershell
   git clone https://github.com/aptana/ArcMap-MCP.git
   cd ArcMap-MCP
   uv sync
   ```

2. Install and enable the companion ArcMap MCP Add-In in ArcMap. Start its MCP server from the Add-In toolbar and keep ArcMap open.

3. Configure your MCP client.

## Configure Visual Studio Code

Create or update `.vscode/mcp.json` in your workspace. Replace the directory with the absolute path to your local clone:

```json
{
  "servers": {
    "arcmap": {
      "type": "stdio",
      "command": "uv",
      "args": [
        "--directory",
        "C:/path/to/ArcMap-MCP",
        "run",
        "arcgis_mcp_server.py"
      ]
    }
  }
}
```

If `uv` is not on the VS Code process's `PATH`, set `command` to the full path to `uv.exe`. Start or restart the MCP server from VS Code after saving the configuration.

## Run Manually

From the repository directory, run:

```powershell
uv run arcgis_mcp_server.py
```

The MCP server uses standard input/output (stdio) for client communication. Separately, it connects to the ArcMap Add-In over TCP on `127.0.0.1:27173`. Set the `ARCGIS_MCP_PORT` environment variable before starting the server if the Add-In is configured to use a different port:

```powershell
$env:ARCGIS_MCP_PORT = "27174"
uv run arcgis_mcp_server.py
```

Use the same port value in the Add-In and in `ARCGIS_MCP_PORT`.

## Available Tools

- **Projects and layers:** `ping`, `get_project_info`, `get_layers`, `add_vector_layer`, `add_raster_layer`, `remove_layer`, `save_project`, `set_layer_visibility`
- **Map navigation:** `get_map_extent`, `zoom_to_layer`, `zoom_to_coordinates`, `get_map_scale`, `set_map_scale`, `get_map_bookmarks`, `apply_bookmark`
- **Data inspection and selection:** `get_layer_schema`, `get_selected_features`, `get_layer_features`, `select_by_attribute`, `set_definition_query`
- **Data frames and layouts:** `list_data_frames`, `switch_data_frame`, `get_layout_elements`
- **Symbology and geoprocessing:** `apply_symbology`, `set_layer_transparency`, `set_layer_color`, `set_layer_labels`, `execute_geoprocessing`
- **Code execution:** `execute_code`

## Compatibility and Safety

- The MCP server requires Python 3.10 or newer. `execute_code` runs scripts using ArcGIS 10.8's Python 2.7 interpreter, so code passed to that tool must use Python 2.7 syntax.
- `execute_code` runs as a standalone process. `arcpy.mapping.MapDocument('CURRENT')` is not available; pass an explicit `.mxd` path instead.
- Tool paths must be accessible to the ArcMap machine. Vector, raster, layer, and project paths should be absolute paths.
- Geoprocessing, layer edits, selection, and code execution can change data or project state. Back up important data and review operations before running them.
- The server connects only to `127.0.0.1`; run it on the same machine as the ArcMap Add-In.

## Troubleshooting

- **Cannot connect to ArcMap:** Confirm ArcMap is open and the Add-In's **Start MCP Server** toolbar button is active.
- **Connection refused:** Confirm both components use the same port. The default is `27173`; `ARCGIS_MCP_PORT` changes the MCP server's destination port.
- **Dependency errors:** Run `uv sync` from the repository root and restart the MCP server.
- **`execute_code` errors:** Use Python 2.7-compatible syntax and explicit `.mxd` paths with `arcpy.mapping.MapDocument`.
