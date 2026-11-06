# REA (Reverse Engineer Anything) Antigravity Plugin & Skills Database

[![GitHub Repository](https://img.shields.io/badge/GitHub-morluto%2Frea-blue?logo=github)](https://github.com/morluto/rea)
[![Antigravity Compatible](https://img.shields.io/badge/Antigravity-Plugin%20v1.0-emerald)](https://github.com/morluto/rea)
[![MCP Enabled](https://img.shields.io/badge/MCP-200%2B%20Tools-purple)](https://modelcontextprotocol.io)

An all-in-one, enterprise-grade **Antigravity Plugin** and modular **Reverse Engineering Skills Database** powered by [REA (Reverse Engineer Anything)](https://github.com/morluto/rea).

This repository provides full Model Context Protocol (MCP) tool integration, specialized autonomous subagents, strict security rules (including prompt-injection defense against malicious binary targets), and a categorized database of reverse-engineering skills spanning native binaries, Electron applications, web bundles, Android APKs, network captures, and firmware.

---

## Architecture & Directory Layout

```text
.
├── plugin.json                                # Antigravity plugin manifest & suggested prompts
├── mcp_config.json                            # REA Model Context Protocol server configuration
├── README.md                                  # User-facing installation guide and documentation
├── rules/
│   └── AGENTS.md                              # Mandatory security, untrusted data, & evidence rules
├── agents/
│   └── rea-reverse-engineer.md                # Specialized autonomous reverse-engineering subagent
└── skills/                                    # Modular Antigravity Skills Database
    ├── reverse-engineer-anything/             # Master orchestration skill (200+ REA tools)
    │   ├── SKILL.md
    │   └── references/
    │       ├── native-and-artifacts.md        # Deep reference: PE, ELF, Mach-O, Go/Rust metadata
    │       ├── javascript-applications.md     # Deep reference: ASAR, minified JS, source maps
    │       ├── android-applications.md        # Deep reference: APK, DEX, JADX decompilation
    │       ├── runtime-observation.md         # Deep reference: Chrome CDP, live browser scenarios
    │       ├── evidence-workflows.md          # Deep reference: Evidence IDs, ledgers, verification
    │       └── connection-and-recovery.md     # Deep reference: REA doctor, setup, engine repair
    ├── rea-native-binary-analysis/            # Focused skill: Ghidra, Hopper, IDA, Kuna, xrefs
    │   └── SKILL.md
    ├── rea-javascript-electron-analysis/      # Focused skill: ASAR unpacking, Wakaru, IPC auditing
    │   └── SKILL.md
    ├── rea-android-apk-analysis/              # Focused skill: JADX, AndroidManifest, Frida hooking
    │   └── SKILL.md
    ├── rea-network-runtime-inspection/        # Focused skill: HAR/SAZ, PCAP/TShark, CDP scenarios
    │   └── SKILL.md
    ├── rea-firmware-hardware-analysis/        # Focused skill: Binwalk/Unblob, WASM, sigrok decoding
    │   └── SKILL.md
    └── rea-evidence-verification/             # Focused skill: Forensic diffing, reconstruction proofs
        └── SKILL.md
```

---

## Modular Skills Database Catalog

| Skill Name | Primary Targets & Scope | Key Tools Used |
| :--- | :--- | :--- |
| **`reverse-engineer-anything`** | Master orchestration & target dispatching across all 200+ REA tools. | `open_binary`, `inspect_artifact`, `binary_session`, `binary_overview` |
| **`rea-native-binary-analysis`** | Windows PE, Linux ELF, macOS Mach-O, Go & Rust build metadata, disassembler engines (Ghidra, Hopper, IDA Pro, Kuna), calling conventions, cross-references (`xrefs`). | `inspect_go_binary`, `inspect_rust_binary`, `inspect_binary_layout`, `analyze_function`, `xrefs` |
| **`rea-javascript-electron-analysis`** | Electron desktop apps, `.asar` packages, minified web bundles, Wakaru source recovery, Electron security fuses, IPC handler auditing. | `analyze_javascript_application`, `recover_javascript_sources`, `inspect_electron_fuses`, `inspect_analysis_view` |
| **`rea-android-apk-analysis`** | Android APKs, AABs, Dalvik/ART bytecode, headless JADX decompilation, `AndroidManifest.xml` auditing, permission tracking, ADB/Frida integration. | `inspect_android_package`, `search_android_classes`, `inspect_android_method`, `trace_android_references` |
| **`rea-network-runtime-inspection`** | Offline HTTP traffic (HAR, SAZ, qlog), Wireshark PCAP/TShark captures, browser CDP scenario automation, live process debugging. | `inspect_web_network_capture`, `inspect_packet_capture`, `inspect_pcap_capture`, `capture_browser_scenario` |
| **`rea-firmware-hardware-analysis`** | IoT firmware images (Binwalk/Unblob carving), WebAssembly (WASM/WAT), EVM smart contract interfaces, digital signal decoding (sigrok). | `inspect_firmware_regions`, `extract_firmware`, `inspect_wasm_artifact`, `inspect_evm_interface`, `decode_signal_recording` |
| **`rea-evidence-verification`** | Immutable Evidence tracking (`evidence_id`), session unknown ledgers, multi-version binary diffing, clean-room reconstruction verification. | `inspect_analysis_view`, `record_unknown`, `compare_artifacts`, `compare_functions`, `verify_reconstruction` |

---

## Installation Guide (Local User)

> [!NOTE]
> All paths in this guide use generic environment variables to protect your privacy and ensure seamless compatibility across machines.

### Prerequisites

1. **Node.js**: Version `22.19.0` or newer (required for running `rea-agents` via `npx`).
   ```bash
   node --version
   ```
2. **Optional Reverse Engineering Engines** (install only as needed for specific targets):
   - **Ghidra / Hopper / IDA Pro**: For native binary decompilation.
   - **Java 17+ & JADX JAR**: For headless Android APK decompilation.
   - **Wireshark / TShark**: For raw `.pcap` / `.pcapng` packet inspection.
   - **Binwalk / Unblob**: For firmware unpacking on Linux.

---

### Option 1: Global Installation (Recommended)

Installing the plugin to your global Antigravity configuration directory makes all REA tools, agents, rules, and skills available across **all** your workspaces.

#### On Windows (PowerShell)

Open PowerShell and copy this plugin repository into your user plugins folder:

```powershell
# Define target destination
$Destination = "$env:USERPROFILE\.gemini\config\plugins\rea-plugin"

# Create target directory if it does not exist
New-Item -ItemType Directory -Path $Destination -Force | Out-Null

# Copy plugin files (execute from this workspace root)
Copy-Item -Path ".\*" -Destination $Destination -Recurse -Force

Write-Host "REA plugin successfully copied to: $Destination" -ForegroundColor Green
```

#### On Windows (Command Prompt)

```cmd
set DEST="%USERPROFILE%\.gemini\config\plugins\rea-plugin"
mkdir %DEST%
xcopy /E /I /Y * %DEST%
```

#### On macOS & Linux (Bash / Zsh)

Open your terminal and copy this plugin repository into your user configuration directory:

```bash
# Define target destination
DEST="$HOME/.gemini/config/plugins/rea-plugin"

# Create destination directory
mkdir -p "$DEST"

# Copy plugin contents
cp -r ./* "$DEST/"

echo "REA plugin successfully copied to: $DEST"
```

---

### Option 2: Project / Workspace-Scoped Installation

If you prefer to keep this plugin scoped only to a specific project:

1. Create a `.gemini/plugins/rea-plugin/` directory in your project's root:
   - Windows (PowerShell):
     ```powershell
     New-Item -ItemType Directory -Path ".\.gemini\plugins\rea-plugin" -Force
     Copy-Item -Path ".\*" -Destination ".\.gemini\plugins\rea-plugin" -Recurse -Force
     ```
   - macOS & Linux:
     ```bash
     mkdir -p ./.gemini/plugins/rea-plugin
     cp -r ./* ./.gemini/plugins/rea-plugin/
     ```

---

## Post-Installation & Verification

1. **Clean Up Conflicting Standalone MCP Entries**:
   If you previously added a standalone `rea` entry to your global `~/.gemini/config/mcp_config.json`, remove it. The plugin provides its own managed `mcp_config.json` to prevent collisions.

2. **Reload Antigravity**:
   Restart Antigravity or reopen your workspace. Antigravity will discover `plugin.json`, load the `rea` MCP server, register the `rea-reverse-engineer` subagent, and activate the skills database.

3. **Verify Connectivity**:
   Run REA's diagnostic doctor command to ensure your environment is fully recognized:
   ```bash
   npx -y rea-agents@latest doctor --json
   ```

4. **Test in Chat**:
   Ask Antigravity:
   > *"Analyze this binary using the REA plugin and summarize its functions."*
   or invoke the specialized subagent:
   > *"@rea-reverse-engineer inspect this ASAR bundle and check for Electron security vulnerabilities."*

---

## Safety, Privacy & Target Data Isolation

- **Prompt Injection Defense**: Target strings, decompiled pseudocode, and symbol names are strictly isolated as untrusted data (`_meta.provenance.untrusted_content`). Embedded malicious prompts within analyzed targets will not be executed by Antigravity agents.
- **Privacy Preservation**: No personal user paths, usernames, or sensitive machine tokens are hardcoded into this plugin or sent over external networks. All analysis is executed locally on your workstation.
- **Summary-First Delivery**: Large binaries and multi-megabyte bundles are summarized first, preventing context overflow and reducing token overhead.
