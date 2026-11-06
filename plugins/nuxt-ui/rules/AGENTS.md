# Nuxt UI Agent Guidelines & Rules

When building, refactoring, or reviewing components and pages that use Nuxt UI (`@nuxt/ui`), adhere strictly to the following architectural rules and conventions:

## 1. Always Verify with the Nuxt UI MCP Server

Never guess component prop names, slot configurations, or event signatures. Nuxt UI evolves across major versions (e.g. Nuxt UI v3 built on Reka UI and Tailwind CSS v4 vs v2 built on Headless UI).

- **Inspect APIs before writing code**:
  - Call `get-component-metadata` (e.g., component `UButton`, `UInput`, `UTable`, `UModal`) to get verified props, slots, and events.
  - Call `get-component` with sections (`usage`, `examples`, `api`, `theme`) when you need full implementation patterns.
  - Call `search-components` if you are unsure which component matches a requested user interface pattern (e.g., segmented controls, comboboxes, drawers).
- **Icons**:
  - Always verify icon names with `search-icons`. Nuxt UI defaults to Lucide icons formatted as `i-lucide-<name>` (e.g., `i-lucide-search`, `i-lucide-arrow-right`, `i-lucide-check`).
- **Composables**:
  - Use `search-composables` to find built-in helpers such as `useToast()`, `useModal()`, and `useSlideover()`.

## 2. Component Auto-Import & Prefixes

- Nuxt UI components are prefixed with `U` (e.g., `<UButton>`, `<UCard>`, `<UModal>`, `<UTable>`, `<UInput>`, `<USelect>`, `<UFormField>`).
- In Nuxt applications, components are automatically imported. **Do not** manually import components from `@nuxt/ui` (such as `import { UButton } from '@nuxt/ui'`) unless in a raw standalone Vue setup without the Nuxt module.

## 3. Component Hierarchy and Forms

- **Form Fields**:
  - In Nuxt UI v3, wrap input elements in `<UFormField>` (providing `name`, `label`, `description`, `help`, and `error` attributes).
  - Form state and validation should pair `<UForm>` with schemas using Zod, Valibot, or Joi via `:schema="schema"` and `:state="state"`.
  - Handle submission with `@submit="onSubmit"`.

## 4. Design System & Styling Conventions

- **Semantic Colors**: Use semantic color names (`primary`, `neutral`, `success`, `warning`, `error`, `info`) rather than raw Tailwind color names (like `green-500` or `blue-600`) for component color props (`color="primary"`).
- **Variants**: Leverage official component variants: `solid`, `outline`, `soft`, `subtle`, `ghost`, and `link`.
- **Customizing Slots & Theme**: Use the `ui` prop on components to target inner sub-elements with Tailwind classes rather than overriding scoped styles or fighting component encapsulation.

## 5. Overlay Management

- Prefer programmatic overlay triggers with composables (`useToast()`, `useModal()`, `useSlideover()`) or v-model binding (`v-model:open="isOpen"`) over manually mutating raw DOM elements.
