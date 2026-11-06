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
├── README.md                                  # Repository overview, catalog list, and global installation instructions
├── .gitignore                                 # Standard Git ignore rules
├── LICENSE                                    # MIT License
└── plugins/                                   # Plugins Catalog
    ├── nuxt/                                  # Official Nuxt 4/3 Framework Integration
    │   ├── plugin.json                        # Plugin manifest & suggested prompts
    │   ├── README.md                          # Documentation & usage guide
    │   ├── rules/AGENTS.md                    # Nuxt SSR & Nitro architectural rules
    │   ├── agents/nuxt-architect.md           # Full-Stack Nuxt Architect subagent
    │   └── skills/nuxt/                       # Nuxt documentation, modules & SSR workflows
    ├── nuxt-ui/                               # Nuxt UI Component Design System
    │   ├── plugin.json                        # Plugin manifest & suggested prompts
    │   ├── README.md                          # Documentation & usage guide
    │   ├── rules/AGENTS.md                    # Nuxt UI v3 & Tailwind CSS conventions
    │   ├── agents/nuxt-ui-architect.md        # Nuxt UI Design Architect subagent
    │   └── skills/nuxt-ui/                    # Live component schemas, icons & form builders
    └── rea/                                   # REA (Reverse Engineer Anything) Suite
        ├── plugin.json                        # Plugin manifest & suggested prompts
        ├── mcp_config.json                    # MCP server definition (rea-agents)
        ├── README.md                          # Comprehensive REA documentation & guides
        ├── rules/AGENTS.md                    # Safety, prompt-injection defense, & evidence rules
        ├── agents/rea-reverse-engineer.md     # Autonomous reverse-engineering subagent
        └── skills/                            # Categorized Skills Database (7 Skills + 6 Guides)
            ├── reverse-engineer-anything/     # Master orchestration & target dispatching
            ├── rea-native-binary-analysis/    # Native binaries (PE, ELF, Mach-O, Ghidra, Hopper, IDA, Kuna)
            ├── rea-javascript-electron-analysis/ # Electron ASAR, minified JS bundles, Wakaru recovery
            ├── rea-android-apk-analysis/      # Android APK/DEX, headless JADX decompilation, Frida
            ├── rea-network-runtime-inspection/ # HAR/SAZ, Wireshark PCAP/TShark, CDP scenarios
            ├── rea-firmware-hardware-analysis/ # Binwalk/Unblob carving, WASM, sigrok logic decoding
            └── rea-evidence-verification/     # Forensic evidence ledgers, unknown tracking, diffing
```

---

## 📦 Plugins Catalog

| Plugin Name | Directory | Description | Integrated MCP Server | Subagents & Skills |
| :--- | :--- | :--- | :--- | :--- |
| **Nuxt** | [`plugins/nuxt/`](plugins/nuxt/) | Official Nuxt framework integration: explore Nuxt 4/3 docs, discover community modules, configure Nitro server routes, and check deployment providers. | Remote Nuxt MCP (`https://nuxt.com/mcp`) | 1 Subagent (`nuxt-architect`)<br>1 Skill (`nuxt`) |
| **Nuxt UI** | [`plugins/nuxt-ui/`](plugins/nuxt-ui/) | Official Nuxt UI integration: discover components, fetch live schemas, search icons, and build Vue/Nuxt interfaces with real-time MCP tooling. | Remote Nuxt UI MCP (`https://ui.nuxt.com/mcp`) | 1 Subagent (`nuxt-ui-architect`)<br>1 Skill (`nuxt-ui`) |
| **REA (Reverse Engineer Anything)** | [`plugins/rea/`](plugins/rea/) | Deep reverse-engineering suite for native binaries, Electron apps, Android APKs, network captures, and firmware. | Local REA MCP (`rea-agents` via `npx`) with 200+ tools | 1 Subagent (`rea-reverse-engineer`)<br>7 Modular Skills + 6 Guides |

---

## 🚀 How to Install Plugins on Local User

> [!NOTE]
> All paths in this guide use standard system environment variables (`$env:USERPROFILE`, `%USERPROFILE%`, `$HOME`, `~`) to ensure privacy and cross-platform compatibility without hardcoding local usernames.

### Prerequisites

1. **Google Antigravity** installed and configured.
2. **Node.js** version `22.19.0` or higher (required for executing MCP tools).
   ```bash
   node --version
   ```

---

### Option A: Install a Specific Plugin

#### Windows (PowerShell)

Run PowerShell and copy the target plugin from this repository into your Antigravity user plugins directory:

```powershell
# Define target plugin name: "nuxt", "nuxt-ui", or "rea"
$PluginName = "rea"

# Target user directory
$Destination = "$env:USERPROFILE\.gemini\config\plugins\$PluginName"

# Create destination directory
New-Item -ItemType Directory -Path $Destination -Force | Out-Null

# Copy plugin contents
Copy-Item -Path ".\plugins\$PluginName\*" -Destination $Destination -Recurse -Force

Write-Host "Plugin '$PluginName' installed successfully to: $Destination" -ForegroundColor Green
```

#### Windows (Command Prompt)

```cmd
set PLUGIN_NAME=rea
set DEST="%USERPROFILE%\.gemini\config\plugins\%PLUGIN_NAME%"

mkdir %DEST%
xcopy /E /I /Y "plugins\%PLUGIN_NAME%\*" %DEST%
```

#### macOS & Linux (Bash / Zsh)

```bash
PLUGIN_NAME="rea"
DEST="$HOME/.gemini/config/plugins/$PLUGIN_NAME"

mkdir -p "$DEST"
cp -r "./plugins/$PLUGIN_NAME/"* "$DEST/"

echo "Plugin '$PluginName' installed successfully to: $DEST"
```

---

### Option B: Install All Plugins at Once

#### Windows (PowerShell)

```powershell
$SourcePlugins = ".\plugins"
$DestBase = "$env:USERPROFILE\.gemini\config\plugins"

Get-ChildItem -Path $SourcePlugins -Directory | ForEach-Object {
    $TargetDir = Join-Path $DestBase $_.Name
    New-Item -ItemType Directory -Path $TargetDir -Force | Out-Null
    Copy-Item -Path (Join-Path $_.FullName "*") -Destination $TargetDir -Recurse -Force
    Write-Host "Installed: $($_.Name)" -ForegroundColor Cyan
}

Write-Host "All plugins successfully installed!" -ForegroundColor Green
```

#### macOS & Linux (Bash / Zsh)

```bash
DEST_BASE="$HOME/.gemini/config/plugins"
mkdir -p "$DEST_BASE"

for plugin in ./plugins/*/; do
    plugin_name=$(basename "$plugin")
    target="$DEST_BASE/$plugin_name"
    mkdir -p "$target"
    cp -r "$plugin"* "$target/"
    echo "Installed: $plugin_name"
done

echo "All plugins successfully installed!"
```

---

## 🔄 Post-Installation & Reloading

1. **Avoid MCP Collisions**: If you previously had a standalone manual entry in your global `mcp_config.json` for an installed plugin, remove it. Antigravity manages each plugin's `mcp_config.json` automatically.
2. **Reload Antigravity**: Restart the Antigravity application or reload the workspace window.
3. **Verify Installation**: Open the Customizations menu in Antigravity to view the newly loaded plugins, active subagents, and skills.

---

## 🛡️ Privacy & Security

- **Zero Local Footprint**: Repository configuration and installation scripts rely strictly on standard relative paths and OS environment variables.
- **Untrusted Target Isolation**: Built-in agent rules enforce that decompiled target code, strings, and network payloads are treated purely as data, preventing prompt-injection attacks.

---

## 📄 License

This repository and its plugin catalog are released under the [MIT License](LICENSE).
