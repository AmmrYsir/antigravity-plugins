# Meshy 3D AI Plugin for Antigravity

Integrates the official **Meshy AI MCP Server** into Antigravity, equipping your AI assistant with end-to-end 3D generation, Smart Topology modeling, PBR texturing, character auto-rigging, animation, and 3D printing preparation.

---

## What's Included

- **28 MCP Tools**: Covering Text-to-3D, Image-to-3D, Multi-Image-to-3D, Retexturing, Smart Topology, Auto-Rigging, Animation, Text-to-Motion, UV Unwrapping, 3D Print Analysis, Multicolor 3MF Processing, Auto-Split, and Slicer Launching.
- **Skill (`meshy-3d`)**: Step-by-step guidance for game assets, rigged characters, 3D printing, and AR exports.
- **Agent (`meshy-3d-specialist`)**: Specialized subagent dedicated to 3D pipeline tasks.
- **Rules (`rules/AGENTS.md`)**: Enforces mandatory user cost confirmation before calling paid endpoints, model selection discipline, and safe task polling.

---

## Configuration & API Key

1. **Obtain an API Key**:
   - Create or access your API key at [https://www.meshy.ai/developers/keys](https://www.meshy.ai/developers/keys) (starts with `msy_`).

2. **Set the Environment Variable**:
   - Set `MESHY_API_KEY` in your system environment variables, OR
   - Edit `mcp_config.json` inside this plugin directory (`~/.gemini/config/plugins/meshy/mcp_config.json`):
     ```json
     {
       "mcpServers": {
         "meshy": {
           "command": "cmd.exe",
           "args": ["/c", "npx", "-y", "@meshy-ai/meshy-mcp-server"],
           "env": {
             "MESHY_API_KEY": "msy_YOUR_ACTUAL_KEY_HERE"
           }
         }
       }
     }
     ```

3. **Restart Antigravity**:
   - Restart Antigravity to discover and register the new plugin and its MCP tools.
   - Manage and toggle the plugin anytime at [Meshy Plugin Settings](customization://meshy).

---

## Starter Prompts

- *"Check my Meshy credit balance and list my recent 3D models"*
- *"Generate a game-ready low-poly knight character in T-pose using Smart Topology and rig it"*
- *"Convert this product image into a 3D model and prepare it for multicolor 3D printing in Bambu Studio"*
