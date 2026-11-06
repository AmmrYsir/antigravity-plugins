# Nuxt Plugin for Antigravity

Integrates the official **Nuxt Framework** Model Context Protocol (MCP) server (`https://nuxt.com/mcp`) into Antigravity to support modern full-stack development with Nuxt 4 and Nuxt 3.

## Features

- **Live MCP Integration**: Connects directly to the official Streamable HTTP MCP server at `https://nuxt.com/mcp`.
- **Nuxt Documentation on Demand**: Browse, search, and retrieve official guides for Nuxt 4, 3, and upcoming versions (`list-documentation-pages`, `get-documentation-page`, `get-getting-started-guide`).
- **Nuxt Modules Ecosystem**: Search, sort, and inspect community and official modules (`list-modules`, `get-module`) with installation instructions and configuration options.
- **Deployment Presets**: Look up deployment targets (Cloudflare, Vercel, Netlify, Docker, Node) and Nitro presets (`list-deploy-providers`, `get-deploy-provider`).
- **Release & Blog Notes**: Access release posts and migration guidance (`list-blog-posts`, `get-blog-post`).
- **Enforced Architectural Rules**: Plain markdown rules in `rules/AGENTS.md` guiding the assistant on Nuxt 4 directory organization (`app/`), SSR-safe data fetching (`useFetch`, `useAsyncData`), shared state with `useState`, and Nitro server handlers (`defineEventHandler`).
- **Dedicated Skill & Architect Agent**: Bundles the `nuxt` skill and the `nuxt-architect` agent.

---

## MCP Server Configuration

Configured in `mcp_config.json`:

```json
{
  "mcpServers": {
    "nuxt": {
      "url": "https://nuxt.com/mcp"
    }
  }
}
```

### Available Tools

| Tool | Purpose |
| :--- | :--- |
| `list-documentation-pages` | List and search Nuxt documentation pages with version filters (`3.x`, `4.x`, `5.x`, `all`). |
| `get-documentation-page` | Retrieve full content and code examples for a specific documentation path. |
| `get-getting-started-guide` | Fetch official setup and initialization instructions for a given version. |
| `list-modules` | List community modules with sorting (downloads, stars, recency) and keyword filtering. |
| `get-module` | Fetch complete details, compatibility, and setup options for a Nuxt module. |
| `list-deploy-providers` | Browse supported hosting targets and platform providers. |
| `get-deploy-provider` | Retrieve deployment setup guidelines and Nitro configuration for a hosting target. |
| `list-blog-posts` | List official Nuxt blog articles and version release announcements. |
| `get-blog-post` | Fetch article contents and release notes by path. |

### Available Context Resources

- `resource://nuxt-com/documentation-pages`
- `resource://nuxt-com/blog-posts`
- `resource://nuxt-com/deploy-providers`

---

## Directory Structure

```text
~/.gemini/config/plugins/nuxt/
├── plugin.json              # Plugin manifest
├── README.md                # Documentation and overview
├── mcp_config.json          # Remote MCP server config (https://nuxt.com/mcp)
├── assets/
│   └── logo.svg             # Nuxt brand vector logo
├── rules/
│   └── AGENTS.md            # Guidelines for Nuxt 4/3 architecture & SSR safety
├── skills/
│   └── nuxt/
│       └── SKILL.md         # Full-stack recipes, data fetching, & Nitro routes
└── agents/
    └── nuxt-architect.md     # Specialized Nuxt Architect agent
```

---

## Activation

Restart Antigravity or open a new session so the newly added plugin directory is discovered. Once loaded, the `nuxt` MCP server, tools, rules, and skills will be active alongside your `nuxt-ui` plugin.
