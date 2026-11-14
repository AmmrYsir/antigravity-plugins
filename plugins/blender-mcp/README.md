# Blender MCP Plugin

Connects Antigravity to Blender 5.2+ using Blender Lab's official `blender-mcp` server (`stdio` transport) and the `MCP` Blender add-on (`localhost:9876`).

## Prerequisites in Blender 5.2

1. **Allow Online Access** enabled in **Edit → Preferences → System → Network**.
2. **MCP** add-on (`Blender Lab`) enabled in **Edit → Preferences → Add-ons / Extensions** with the bridge server running (default `localhost:9876`).
3. Keep Blender 5.2 open when using interactive scene, viewport, or screenshot tools.

## Environment Overrides

Edit `mcp_config.json` `"env"` if needed:
- `BLENDER_PATH`: Path to `blender.exe` for `*_for_cli` background tools (configured for Blender 5.2).
- `BLENDER_MCP_HOST`: Bridge socket host (default `localhost`).
- `BLENDER_MCP_PORT`: Bridge socket port (default `9876`).
