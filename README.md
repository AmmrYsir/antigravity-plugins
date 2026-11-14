# Antigravity Plugins Catalog

[![Antigravity](https://img.shields.io/badge/Google-Antigravity-4285F4?logo=google)](https://github.com/AmmrYsir/antigravity-plugins)
[![Plugins](https://img.shields.io/badge/Catalog-Plugins%20%26%20Skills-emerald)](plugins/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Welcome to the **Antigravity Plugins Catalog** — a curated repository of enterprise-grade plugins, specialized subagents, Model Context Protocol (MCP) toolchains, and modular skills databases designed for Google Antigravity.

---

## 📂 Repository Structure

Each plugin in this repository is self-contained under the `plugins/` directory:

```text
antigravity-plugins/
├── README.md                                  # Repository overview, catalog list, and installation guide
├── .gitignore                                 # Standard Git ignore rules
├── LICENSE                                    # MIT License
└── plugins/                                   # Plugins Catalog
    ├── blender-mcp/                           # Blender 5.2 MCP integration (scene inspection, rendering, Python)
    ├── meshy/                                 # Meshy 3D AI generation, Smart Topology, rigging, & printing
    ├── nuxt/                                  # Official Nuxt framework integration
    ├── nuxt-ui/                               # Official Nuxt UI component & schema integration
    ├── rea/                                   # REA (Reverse Engineer Anything) Suite
    │   ├── plugin.json                        # Plugin manifest & suggested prompts
    │   ├── mcp_config.json                    # MCP server definition (rea-agents)
    │   ├── README.md                          # Comprehensive REA usage & workflow guide
    │   ├── rules/AGENTS.md                    # Safety, prompt-injection defense, & evidence rules
    │   ├── agents/rea-reverse-engineer.md     # Autonomous reverse-engineering subagent
    │   └── skills/                            # Categorized Skills Database (7 Skills + 6 Guides)
    │       ├── reverse-engineer-anything/     # Master orchestration & target dispatching
    │       ├── rea-native-binary-analysis/    # Native binaries (PE, ELF, Mach-O, Ghidra, Hopper, IDA, Kuna)
    │       ├── rea-javascript-electron-analysis/ # Electron ASAR, minified JS bundles, Wakaru recovery
    │       ├── rea-android-apk-analysis/      # Android APK/DEX, headless JADX decompilation, Frida
    │       ├── rea-network-runtime-inspection/ # HAR/SAZ, Wireshark PCAP/TShark, CDP scenarios
    │       ├── rea-firmware-hardware-analysis/ # Binwalk/Unblob carving, WASM, sigrok logic decoding
    │       └── rea-evidence-verification/     # Forensic evidence ledgers, unknown tracking, diffing
    └── roblox-game-plugin/                    # Roblox game development, Luau architecture, & Studio building
```

---

## 📦 Plugins Catalog

| Plugin Name | Directory | Description | Integrated MCP Server | Subagents & Skills |
| :--- | :--- | :--- | :--- | :--- |
| **Blender MCP** | [`plugins/blender-mcp/`](plugins/blender-mcp/) | Connects Antigravity to Blender 5.2 via Blender Lab's MCP server for scene inspection, screenshots, rendering, and Python execution. | Local Blender MCP (`blender-mcp.exe`) | — |
| **Meshy 3D AI** | [`plugins/meshy/`](plugins/meshy/) | Meshy AI 3D generation platform: text-to-3D, image-to-3D, Smart Topology, PBR texturing, character auto-rigging & animation, and 3D printing preparation. | Local Meshy MCP (`@meshy-ai/meshy-mcp-server` via `npx`) | 1 Subagent (`meshy-3d-specialist`)<br>1 Skill (`meshy-3d`) |
| **Nuxt** | [`plugins/nuxt/`](plugins/nuxt/) | Official Nuxt framework integration: explore Nuxt 4/3 docs, discover community modules, configure Nitro server routes, and check deployment providers. | Remote Nuxt MCP (`https://nuxt.com/mcp`) | 1 Subagent (`nuxt-architect`)<br>1 Skill (`nuxt`) |
| **Nuxt UI** | [`plugins/nuxt-ui/`](plugins/nuxt-ui/) | Official Nuxt UI integration: discover components, fetch live schemas, search icons, and build Vue/Nuxt interfaces with real-time MCP tooling. | Remote Nuxt UI MCP (`https://ui.nuxt.com/mcp`) | 1 Subagent (`nuxt-ui-architect`)<br>1 Skill (`nuxt-ui`) |
| **REA (Reverse Engineer Anything)** | [`plugins/rea/`](plugins/rea/) | Deep reverse-engineering suite for native binaries, Electron apps, Android APKs, network captures, and firmware. | Local REA MCP (`rea-agents` via `npx`) with 200+ tools | 1 Subagent (`rea-reverse-engineer`)<br>7 Modular Skills + 6 Guides |
| **Roblox Game Development** | [`plugins/roblox-game-plugin/`](plugins/roblox-game-plugin/) | Comprehensive Roblox game development suite with Luau architecture patterns, game scaffolds, security hardening, monetization auditing, and Studio integration. | Roblox Studio MCP integration | 1 Subagent (`roblox-studio-builder`)<br>2 Skills (`roblox-game`, `roblox-studio-building`) |

---

## 🚀 How to Install Plugins

Installing any plugin in Antigravity is as simple as copying the plugin folder into your Antigravity user plugins directory.

### Step 1: Copy the plugin folder

Copy any plugin folder you want from `plugins/` (for example, `plugins/rea`, `plugins/nuxt`, or `plugins/nuxt-ui`) into:

📁 **`.gemini` ➔ `config` ➔ `plugins`** in your user home directory:

- **Windows**: `%USERPROFILE%\.gemini\config\plugins\` (or `C:\Users\<your-username>\.gemini\config\plugins\`)
- **macOS / Linux**: `~/.gemini/config/plugins/` (or `$HOME/.gemini/config/plugins/`)

*(If the `plugins` folder does not exist yet inside `.gemini/config/`, simply create it.)*

### Step 2: Restart Antigravity

Restart Antigravity (or reopen your workspace). Antigravity will automatically detect the plugin, register its MCP server, and activate its subagents and skills.

### Step 3: That's it!

Your plugin is now active! You can view, enable, or disable installed plugins anytime from the **Customizations** page in Antigravity.

---

## 🛡️ Privacy & Security

- **Zero Local Footprint**: Repository configuration and instructions rely strictly on relative paths and user home placeholders (`~/.gemini/` or `%USERPROFILE%`), keeping your local environment completely private.
- **Untrusted Target Isolation**: Built-in agent rules enforce that decompiled target code, strings, and network payloads are treated purely as inert data, preventing prompt-injection attacks.

---

## 📄 License

This repository and its plugin catalog are released under the [MIT License](LICENSE).
