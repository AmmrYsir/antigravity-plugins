---
name: nuxt
description: Build, configure, and maintain modern Nuxt applications (Nuxt 4 and 3). Covers consulting the official Nuxt MCP server, data fetching, Nitro server routes, state management, module integration, and deployment configurations.
---

# Nuxt Skill

This skill guides you through developing, configuring, and architecting full-stack applications with **Nuxt** (Nuxt 4 and Nuxt 3). It details how to leverage the official Nuxt Model Context Protocol (MCP) server (`https://nuxt.com/mcp`) to retrieve verified documentation, select community modules, implement server routes, and prepare production deployments.

---

## When to Use This Skill

Activate this skill when:
- Creating or configuring a Nuxt application (`nuxt.config.ts`, Nitro options, runtime config).
- Structuring directories according to Nuxt 4 conventions (`app/` vs root `server/` and `public/`).
- Implementing SSR data fetching (`useFetch`, `useAsyncData`) and state management (`useState`).
- Creating backend endpoints and middleware with the Nitro engine (`server/api/`, `server/routes/`).
- Querying Nuxt documentation, modules, and deployment guides via the `nuxt` MCP server.
- Configuring hosting presets (Cloudflare Pages/Workers, Vercel, Netlify, Docker).

---

## MCP Server Integration & Tools

The plugin connects directly to `https://nuxt.com/mcp`. Always use these tools to inspect real-time documentation and avoid outdated information:

### 1. `list-documentation-pages`
List all documentation pages with version and category filters:
- **Input**: `{ "version": "4.x", "search": "data fetching" }` (or `{ "version": "all" }`).
- **Output**: Ranked documentation topics, categories, and paths.

### 2. `get-documentation-page`
Fetch the complete content and code examples for a specific documentation path:
- **Input**: `{ "path": "/docs/4.x/getting-started/data-fetching" }`.
- **Output**: Full guide markdown, code snippets, and API references.

### 3. `get-getting-started-guide`
Retrieve the official scaffolding guide for a specific version:
- **Input**: `{ "version": "4.x" }`.
- **Output**: Step-by-step setup, prerequisites, and starter templates.

### 4. `list-modules`
Search and filter community modules:
- **Input**: `{ "search": "supabase", "sort": "downloads" }`.
- **Output**: Module names, descriptions, maintainers, and install metrics.

### 5. `get-module`
Inspect a specific module's documentation and configuration schema:
- **Input**: `{ "name": "@nuxtjs/tailwindcss" }`.
- **Output**: README, options, compatibility table, and setup instructions.

### 6. `list-deploy-providers` & `get-deploy-provider`
Discover supported deployment providers and hosting configurations:
- **Input**: `{ "path": "cloudflare" }` or `{ "path": "vercel" }`.
- **Output**: Nitro presets, environment variables, and build commands.

---

## MCP Context Resources

You can reference official Nuxt resources using standard URI syntax:
- `resource://nuxt-com/documentation-pages`: Complete directory of documentation pages.
- `resource://nuxt-com/blog-posts`: Nuxt release notes and roadmap announcements.
- `resource://nuxt-com/deploy-providers`: Directory of supported deployment targets.

---

## Core Nuxt Architecture Workflows

### Workflow A: Bootstrapping a Nuxt 4 Project

Initialize a new project using `nuxi`:

```bash
npx nuxi@latest init my-nuxt-app
```

Recommended Nuxt 4 directory structure:
```text
my-nuxt-app/
├── app/
│   ├── app.vue
│   ├── router.options.ts
│   ├── components/
│   ├── composables/
│   ├── layouts/
│   ├── middleware/
│   └── pages/
├── server/
│   ├── api/
│   ├── routes/
│   └── utils/
├── public/
├── nuxt.config.ts
├── package.json
└── tsconfig.json
```

In `nuxt.config.ts`:
```typescript
export default defineNuxtConfig({
  compatibilityDate: '2025-01-01',
  future: {
    compatibilityVersion: 4
  },
  runtimeConfig: {
    apiSecret: '', // Private server-only key
    public: {
      apiBase: '/api' // Exposed to client
    }
  }
})
```

### Workflow B: SSR Data Fetching with `useFetch`

```vue
<script setup lang="ts">
interface Post {
  id: number
  title: string
  content: string
}

// Automatically creates a unique key and manages SSR cache transfer
const { data: posts, status, error, refresh } = await useFetch<Post[]>('/api/posts', {
  lazy: false,
  server: true,
  transform: (data) => data.slice(0, 10)
})
</script>

<template>
  <main class="container mx-auto p-4">
    <div v-if="status === 'pending'" class="text-gray-500">Loading posts...</div>
    <div v-else-if="error" class="text-red-500">Failed to load posts: {{ error.message }}</div>
    <ul v-else class="space-y-2">
      <li v-for="post in posts" :key="post.id" class="p-3 border rounded">
        <h2 class="font-bold">{{ post.title }}</h2>
        <p>{{ post.content }}</p>
      </li>
    </ul>
    <button class="mt-4 px-4 py-2 bg-blue-600 text-white rounded" @click="() => refresh()">
      Refresh
    </button>
  </main>
</template>
```

### Workflow C: Type-Safe Nitro API Handlers

Create `server/api/users/[id].ts`:

```typescript
export default defineEventHandler(async (event) => {
  const id = getRouterParam(event, 'id')
  const query = getQuery(event)

  if (!id) {
    throw createError({
      statusCode: 400,
      statusMessage: 'User ID is required'
    })
  }

  // Example database lookup or external request
  return {
    id,
    role: query.role ?? 'member',
    fetchedAt: new Date().toISOString()
  }
})
```

### Workflow D: SSR-Safe Shared State

```typescript
// app/composables/useAuth.ts
export const useAuth = () => {
  const user = useState<string | null>('auth-user', () => null)

  const login = (username: string) => {
    user.value = username
  }

  const logout = () => {
    user.value = null
  }

  return {
    user,
    login,
    logout
  }
}
```
