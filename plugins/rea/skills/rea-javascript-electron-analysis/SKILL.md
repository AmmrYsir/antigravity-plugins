---
name: rea-javascript-electron-analysis
description: Reverse engineer Electron apps and bundled JavaScript applications using REA. Unpack ASAR archives, decompile minified bundles, recover source code via Wakaru, audit Electron security fuses, trace IPC handlers, and map module dependencies.
---

# JavaScript & Electron Application Analysis with REA

This skill provides step-by-step methodologies for investigating Electron desktop applications, minified web applications, Node.js packages, and client-side JavaScript bundles without original source repositories.

---

## 1. Electron Security & Fuse Auditing

Before diving into JavaScript code, inspect the compiled Electron binary's embedded configuration fuses.

### Check Security Fuses
```json
{
  "path": "/path/to/electron_executable"
}
```
- Tool: `inspect_electron_fuses`
- **Critical Fuses to Evaluate**:
  - `RunAsNode`: If enabled (`true`), an attacker can launch the executable with `ELECTRON_RUN_AS_NODE=1` to run arbitrary Node.js scripts as a local shell.
  - `EnableNodeCliInspectArguments`: If enabled (`true`), the application accepts `--inspect` flags, allowing runtime debugging and arbitrary code execution in production.
  - `EnableEmbeddedAsarIntegrityValidation`: Verifies whether the ASAR payload is cryptographically validated at startup.
  - `OnlyLoadAppFromAsar`: Enforces that only packaged ASAR files can be executed.

---

## 2. ASAR & Application Inspection Workflow

### Step 1: Summary-First Bundle Analysis
Analyze an unpacked directory or `.asar` archive:
```json
{
  "path": "/path/to/resources/app.asar",
  "detail": "summary"
}
```
- Tool: `analyze_javascript_application`
- **Output**: Returns an initial Evidence summary containing:
  - `parent_evidence_id`: Reference handle for the retained in-memory graph.
  - Package metadata (`package.json` dependencies, entry scripts).
  - Detected framework (React, Vue, Angular, Svelte, Electron main/renderer).
  - Module inventory (file count, chunk count, external vs internal imports).

### Step 2: Target Path Filtering
When focusing on specific critical files (e.g. entry points and preload scripts), avoid analyzing entire `node_modules` trees by providing explicit `source_paths`:
```json
{
  "path": "/path/to/resources/app.asar",
  "source_paths": [
    "package.json",
    "main.js",
    "preload.js",
    "src/background.js"
  ]
}
```

### Step 3: Paged Subgraph Exploration
Retrieve specific modules from the retained analysis graph without repeating analysis:
```json
{
  "source": {
    "evidence_id": "<parent_evidence_id>"
  },
  "view": {
    "kind": "modules",
    "scope": "item",
    "node_id": "main.js"
  }
}
```
- Tool: `inspect_analysis_view`

---

## 3. Source Recovery (Wakaru Decompiler)

For heavily bundled and minified bundles (Webpack, Vite, esbuild, Rollup), use REA's source recovery pipeline:

```json
{
  "path": "/path/to/bundle.min.js",
  "additional_paths": [
    "/path/to/vendor.min.js",
    "/path/to/bundle.min.js.map"
  ]
}
```
- Tool: `recover_javascript_sources`
- Decompiles minified bundles back to structured, formatted JavaScript.
- Restores original filenames and directory hierarchies when source maps are present or reconstructible.

---

## 4. Auditing IPC Boundaries in Electron

Electron security centers on the boundary between the untrusted Renderer process (web context) and the privileged Main process (OS access).

1. **Renderer / Preload Audit**:
   - Inspect `preload.js` for calls to `contextBridge.exposeInMainWorld()`.
   - Look for unsafe exposure of `ipcRenderer.send`, `ipcRenderer.invoke`, or raw Node `fs`/`child_process` modules.
2. **Main Process Handlers**:
   - Search for `ipcMain.on(...)` and `ipcMain.handle(...)` registrations in `main.js`.
   - Trace where arguments passed from the renderer are processed. Check for command injection (`exec`, `spawn`), arbitrary file writes, or protocol tampering.
3. **Trace Semantics**:
   - Use `trace_javascript_semantics` and `trace_web_module_imports` to track sensitive data flows across asynchronous IPC dispatches.

---

## 5. Version & Export Shape Diffing

When tracking API changes or malware implants between application versions:
```json
{
  "left_path": "/path/to/v1.0.0/app.asar",
  "right_path": "/path/to/v1.1.0/app.asar"
}
```
- Tool: `compare_javascript_export_shapes`
- Detects newly added, removed, or mutated exports, prototype methods, and exported classes.
