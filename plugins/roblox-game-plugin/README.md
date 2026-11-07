# Roblox Game Development Plugin for Antigravity

An expert-grade game development plugin for Antigravity and Gemini, engineered for building, auditing, optimizing, and deploying Roblox games.

## Features

- **Comprehensive Game Scaffolds**:
  - Simulators (Pet Simulator / clicker style)
  - Tycoons (automated droppers, conveyor systems, plot claiming)
  - RPGs (combat, stats, inventory, questing)
  - Obbies (checkpoint management, stage progression)
  - Horror (dynamic AI stalker, procedural dread, soundscapes)
  - Battle Royale (safe zone shrinkage, loot crates, match loops)
- **Deep Reference System**:
  - Luau mastery & idiomatic typing
  - ProfileService & session-locking persistence
  - Combat networking, client prediction, and hit verification
  - UI/UX frameworks, responsive scaling, controller support
  - Performance profiling, streaming optimizations, draw calls
  - Security audits, remote event sanitization, server authority
- **Autonomous MCP Studio Workflows**:
  - Dynamic discovery of Roblox Studio MCP capabilities
  - Live script creation, DataModel manipulation, and debugging loops
- **Embedded Rules (`rules/AGENTS.md`)**:
  - Enforces server-authoritative logic and security standards during every interaction.

## Directory Structure

```text
roblox-game-plugin/
├── plugin.json
├── README.md
├── assets/
│   └── logo.svg
├── rules/
│   └── AGENTS.md
└── skills/
    └── roblox-game/
        ├── SKILL.md
        ├── references/
        ├── templates/
        └── workflows/
```

## How to Install into Antigravity Globally

To make this plugin available globally across all your projects in Antigravity:

1. Copy the `roblox-game-plugin` folder to:
   ```powershell
   Copy-Item -Path "c:\Users\ammar\Desktop\roblox\roblox-game-plugin" -Destination "$HOME\.gemini\config\plugins\roblox-game-plugin" -Recurse
   ```
2. Restart Antigravity to discover the new plugin directory.
