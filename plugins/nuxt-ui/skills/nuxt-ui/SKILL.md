---
name: nuxt-ui
description: Build, design, and maintain Vue & Nuxt applications using Nuxt UI. Covers querying the remote Nuxt UI MCP server for real-time component schemas, props, slots, Lucide icons, composables, and crafting production-ready accessible interfaces.
---

# Nuxt UI Skill

This skill guides you through developing interfaces with **Nuxt UI** (`@nuxt/ui`). It teaches you how to leverage the live Nuxt UI Model Context Protocol (MCP) server (`https://ui.nuxt.com/mcp`) to inspect real-time documentation, retrieve exact component props and slots, search icon sets, and scaffold UI components.

---

## When to Use This Skill

Activate this skill when:
- Designing or implementing UI components in a Vue 3 / Nuxt 3+ application.
- Choosing or configuring Nuxt UI components (`UButton`, `UTable`, `UModal`, `UForm`, `UInput`, etc.).
- Verifying component APIs, props, slots, or events against the live Nuxt UI MCP server.
- Searching for icons using Iconify/Lucide in Nuxt UI's `i-lucide-*` format.
- Configuring Nuxt UI theming, color tokens, and Tailwind CSS v4 styling.

---

## MCP Server Integration & Tools

The plugin connects directly to `https://ui.nuxt.com/mcp`. Always utilize these MCP tools to prevent hallucinations and ensure syntax aligns with the current release:

### 1. `search-components`
Use this tool when finding the right component for a UI requirement or mapping a conceptual element (e.g., "segmented control", "combobox", "drawer"):
- **Input**: `{ "query": "table" }` or leave blank to list all available components.
- **Output**: Ranked components matching the description, category, or keyword.

### 2. `get-component-metadata`
Use this tool to get lightweight, high-precision technical specs without polluting the context window:
- **Input**: `{ "name": "UInput" }` (or `{ "name": "UTable", "full": true }`).
- **Output**: Props list with types and defaults, slot definitions, and emitted events.

### 3. `get-component`
Use this tool when full documentation and code examples are required:
- **Input**: `{ "name": "UModal", "sections": ["usage", "examples", "api"] }`.
- Available sections: `usage`, `examples`, `api`, `theme`, `changelog`.

### 4. `search-icons`
Use this tool whenever selecting an icon for a component (`icon`, `leading-icon`, `trailing-icon`):
- **Input**: `{ "query": "calendar" }`.
- **Output**: Icon names formatted as `i-lucide-<name>` (e.g. `i-lucide-calendar`, `i-lucide-calendar-days`).

### 5. `search-composables`
Use this tool to discover and inspect Nuxt UI composables:
- **Input**: `{ "query": "toast" }` or `{ "query": "modal" }`.
- **Output**: Documentation on composables like `useToast()`, `useModal()`, `useSlideover()`.

---

## Step-by-Step Implementation Workflows

### Workflow A: Adding a Nuxt UI Component to a Page

1. **Discover Component**: Run `search-components` with your use case.
2. **Verify Props & Slots**: Run `get-component-metadata` for the component (e.g., `UTable`).
3. **Verify Icons**: If the component has icon slots or props, run `search-icons` to find appropriate Lucide icon strings (`i-lucide-*`).
4. **Draft Component Template**:
   - Nuxt components are auto-imported. Do NOT write explicit script imports.
   - Use semantic colors (`color="primary"`, `color="neutral"`).
   - Use the `ui` prop for sub-element styling.

### Workflow B: Building Validated Forms

Nuxt UI v3 pairs `<UForm>` with schema validators like Zod:

```vue
<script setup lang="ts">
import { z } from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'

const schema = z.object({
  email: z.string().email('Invalid email address'),
  password: z.string().min(8, 'Password must be at least 8 characters')
})

type Schema = z.output<typeof schema>

const state = reactive<Partial<Schema>>({
  email: '',
  password: ''
})

const toast = useToast()

async function onSubmit(event: FormSubmitEvent<Schema>) {
  toast.add({
    title: 'Success',
    description: `Logged in as ${event.data.email}`,
    color: 'success',
    icon: 'i-lucide-check-circle'
  })
}
</script>

<template>
  <UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
    <UFormField label="Email" name="email" required>
      <UInput v-model="state.email" icon="i-lucide-mail" placeholder="you@example.com" class="w-full" />
    </UFormField>

    <UFormField label="Password" name="password" required>
      <UInput v-model="state.password" type="password" icon="i-lucide-lock" class="w-full" />
    </UFormField>

    <UButton type="submit" label="Sign In" color="primary" block />
  </UForm>
</template>
```

### Workflow C: Responsive Data Tables

For tabular data with pagination and sorting:
1. Query `get-component-metadata` for `UTable`.
2. Define column configurations with headers and accessor keys.
3. Use template slots for custom cell rendering (`#<column>-cell="{ row }"`).

---

## Nuxt UI Theming & Configuration

- **`app.config.ts`**: Customize global defaults, primary colors, and component variants:
  ```typescript
  export default defineAppConfig({
    ui: {
      colors: {
        primary: 'emerald',
        neutral: 'zinc'
      }
    }
  })
  ```
- **CSS Setup**: Nuxt UI v3 uses Tailwind CSS v4:
  ```css
  /* assets/css/main.css */
  @import "tailwindcss";
  @import "@nuxt/ui";
  ```
