# Nuxt UI Plugin for Antigravity

Integrates official **Nuxt UI** (`@nuxt/ui`) tooling directly into Antigravity via the remote **Nuxt UI Model Context Protocol (MCP)** server.

## Features

- **Live MCP Connection**: Connects to the official Streamable HTTP MCP server at `https://ui.nuxt.com/mcp`.
- **Component Discovery & Metadata**: Query real-time component schemas, props, slots, and events (`search-components`, `get-component-metadata`, `get-component`) to avoid outdated code generation or hallucinations.
- **Icon Search**: Instantly look up Lucide/Iconify icon names formatted for Nuxt UI (`search-icons`).
- **Composables Reference**: Discover and inspect built-in composables like `useToast`, `useModal`, and `useSlideover` (`search-composables`).
- **Enforced Architectural Rules**: Bundles active rules in `rules/AGENTS.md` guiding the assistant on Nuxt UI v3 conventions (Reka UI, Tailwind CSS v4, `<UForm>`, schema validation with Zod, and semantic color tokens).
- **Specialized Architect Agent & Skill**: Includes the `nuxt-ui` skill and `nuxt-ui-architect` subagent.

---

## Bundled MCP Server

This plugin configures the remote Streamable HTTP MCP server:

```json
{
  "mcpServers": {
    "nuxt-ui": {
      "url": "https://ui.nuxt.com/mcp"
    }
  }
}
```

### Available MCP Tools

| Tool | Purpose |
| :--- | :--- |
| `search-components` | Search components by name, keyword, or intent (or list all when empty). |
| `get-component-metadata` | Retrieve lightweight, exact technical specs: props, slots, and events. |
| `get-component` | Retrieve full documentation and code examples with section filtering (`usage`, `examples`, `api`, `theme`). |
| `search-icons` | Search Iconify collections (default Lucide) for icon strings formatted as `i-lucide-<name>`. |
| `search-composables` | Look up composables and overlay helpers. |

### Available MCP Resources

- `resource://nuxt-ui/components`
- `resource://nuxt-ui/composables`
- `resource://nuxt-ui/examples`
- `resource://nuxt-ui/templates`
- `resource://nuxt-ui/documentation-pages`

---

## Directory Structure

```text
~/.gemini/config/plugins/nuxt-ui/
├── plugin.json                 # Plugin manifest
├── README.md                   # Plugin overview and documentation
├── mcp_config.json             # Remote MCP server config (https://ui.nuxt.com/mcp)
├── assets/
│   └── logo.svg                # Vector logo
├── rules/
│   └── AGENTS.md               # Guidelines & rules for Nuxt UI v3
├── skills/
│   └── nuxt-ui/
│       └── SKILL.md            # Actionable workflows, Zod forms, & component recipes
└── agents/
    └── nuxt-ui-architect.md    # Dedicated Nuxt UI Architect agent
```

---

## Activation

Restart Antigravity or start a new session for the newly created plugin directory to be discovered. Once loaded, the `nuxt-ui` MCP server and all bundled rules and skills will be available.
