# Nuxt Agent Guidelines & Rules

When building, refactoring, or reviewing Nuxt applications (Nuxt 4 and Nuxt 3), adhere strictly to the following framework conventions and architectural rules:

## 1. Always Verify with the Nuxt MCP Server

Never guess Nuxt configuration keys, module names, or composable signatures. Utilize the official `nuxt` MCP server (`https://nuxt.com/mcp`):

- **Documentation**: Call `list-documentation-pages` to search for relevant guides by category and version (default `4.x`), and call `get-documentation-page` with the path to inspect complete examples and API parameters.
- **Modules**: When adding features (e.g., authentication, image optimization, pinia, tailwindcss, content, i18n), query `list-modules` and `get-module` to confirm official package names, installation steps, and configuration options rather than guessing packages.
- **Deployments**: Call `list-deploy-providers` and `get-deploy-provider` when configuring hosting, edge functions, or server presets (Cloudflare, Vercel, Netlify, Node server).
- **Getting Started**: Call `get-getting-started-guide` for bootstrapping workflows and version requirements.

## 2. Directory Structure Conventions

- **Nuxt 4 Structure**:
  - Frontend application code lives in the `app/` directory:
    - `app/app.vue`, `app/router.options.ts`
    - `app/components/`, `app/composables/`, `app/layouts/`, `app/middleware/`, `app/pages/`, `app/plugins/`, `app/utils/`
  - Server logic, public assets, and build modules remain at project root:
    - `server/api/`, `server/routes/`, `server/middleware/`, `server/utils/`
    - `public/`, `modules/`, `nuxt.config.ts`
- In Nuxt 3 projects using `future: { compatibilityVersion: 4 }`, uphold the `app/` layout.

## 3. Data Fetching & SSR Safety

- **Primary Composables**:
  - Use `useFetch(url)` for declarative SSR data fetching with automatic caching and key generation.
  - Use `useAsyncData(key, handler)` for complex multi-source queries, RPC calls, or non-HTTP promises.
- **Avoid Client-Only / Uncached Fetching**:
  - Never call raw `fetch()` or `axios` at the root of a setup script or page component; doing so triggers duplicate executions on both server and client, causing hydration mismatches.
- **State Management**:
  - Use `useState<T>(key, init)` for SSR-safe shared state. Avoid module-level `const shared = ref()` because module-level variables leak across requests in Node/serverless runtimes.

## 4. Server Engine (Nitro)

- Define server endpoints in `server/api/` or `server/routes/` wrapped with `defineEventHandler`:
  - Read query params: `getQuery(event)`
  - Read request body: `await readBody(event)`
  - Throw typed errors: `throw createError({ statusCode: 400, statusMessage: '...' })`
- Place shared backend utilities in `server/utils/` where Nitro auto-imports them.

## 5. Auto-Imports & Clean Imports

- Nuxt auto-imports Vue reactives (`ref`, `computed`, `reactive`, `watch`, `onMounted`), Nuxt composables (`useRoute`, `useRouter`, `useFetch`, `useAsyncData`, `useState`, `useRuntimeConfig`), and project components.
- Avoid redundant explicit imports of these auto-imported APIs in Vue components and composables.
