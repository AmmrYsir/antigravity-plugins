---
name: rea-network-runtime-inspection
description: Network traffic forensics and dynamic runtime observation with REA. Inspect HTTP captures (HAR, SAZ, qlog), analyze raw PCAP packet captures via TShark, automate browser CDP scenarios, and attach Frida/GDB debug sessions.
---

# Network & Runtime Inspection with REA

This skill provides workflows for analyzing recorded network traffic, capturing runtime browser interactions via the Chrome DevTools Protocol (CDP), and executing dynamic debugging sessions.

---

## 1. Network Traffic Forensics

REA inspects saved network artifacts offline without re-fetching remote endpoints or disclosing user credentials.

### HTTP/HTTPS Web Captures (HAR, SAZ, Qlog)
```json
{
  "path": "/path/to/traffic.saz",
  "format": "saz",
  "detail": "summary"
}
```
- Tool: `inspect_web_network_capture`
- **Supported Formats**:
  - `har`: Standard HTTP Archive format from browser DevTools.
  - `saz`: Fiddler Everywhere session archives.
  - `mitmproxy`: Native flow captures from mitmproxy.
  - `qlog`: HTTP/3 and QUIC event traces (aioquic format).
- **Extracted Evidence**: Request/response headers, status codes, query strings, body sizes, timing waterfall, and protocol version. Credential headers (e.g. `Authorization`, `Cookie`) are sanitized by default.

### Raw Packet Capture (PCAP / PCAPNG with TShark)
For transport-layer protocol analysis (TCP, UDP, TLS handshakes, DNS):

1. **Discover Available Protocol Fields**:
   ```json
   {
     "query": "tls.handshake.extensions_server_name"
   }
   ```
   - Tool: `inspect_packet_fields`

2. **Filter and Page Packet Captures**:
   ```json
   {
     "path": "/path/to/capture.pcapng",
     "display_filter": "tcp.port == 443 and tls.handshake.type == 1",
     "fields": [
       "frame.number",
       "ip.src",
       "ip.dst",
       "tls.handshake.extensions_server_name"
     ],
     "match_limit": 50
   }
   ```
   - Tool: `inspect_packet_capture`
   - Applies Wireshark display filter syntax and extracts specified packet fields without loading huge captures into memory.

3. **High-Level PCAP Traffic Summary**:
   ```json
   {
     "path": "/path/to/capture.pcap",
     "detail": "summary"
   }
   ```
   - Tool: `inspect_pcap_capture`
   - Summarizes conversations, DNS query resolutions, and SNI hostnames.

---

## 2. Browser Runtime Automation via CDP

REA can attach to local Chromium-based browsers or launch isolated headless browser scenarios.

### Passive Page Observation
Discover currently open browser tabs and inspect DOM / network:
```json
// List open browser pages on local CDP endpoint
list_browser_targets({
  "endpoint": "http://127.0.0.1:9222"
})

// Passively inspect selected page state
inspect_web_page({
  "endpoint": "http://127.0.0.1:9222",
  "target_id": "<target_id>",
  "include_accessibility_tree": true,
  "include_storage_keys": true
})
```

### Active Scenario Automation
When an investigation requires reproducing specific user actions (e.g., login flow, form submission, SPA route navigation):
```json
{
  "mode": "launch",
  "start_url": "https://example.local/login",
  "actions": [
    { "type": "fill", "selector": "#username", "value": "testuser" },
    { "type": "fill", "selector": "#password", "value": "password123" },
    { "type": "click", "selector": "button[type='submit']" },
    { "type": "wait", "selector": ".dashboard-container", "timeout_ms": 5000 }
  ],
  "capture_screenshot": true
}
```
- Tool: `capture_browser_scenario`
- Launches an ephemeral browser session, executes the declared steps in order, and returns final DOM, network log, and screenshot evidence.

---

## 3. Dynamic Process Debugging (Frida & GDB)

For live native binaries and instrumented processes:

### Frida Instrumentation
- `start_frida_session`: Attach to a running process by name or PID.
- `load_frida_script`: Inject JavaScript hooks to trace arguments, replace implementations, or log memory pointers.
- `close_frida_session`: Detach cleanly.

### GDB & Rizin Debugging
- `start_gdb_session`: Launch or attach a GDB controller for Linux binaries.
- `start_rizin_debug_session`: Start a Rizin/Cutter debugging session.
- `rizin_debug_command`: Send low-level debugger commands (`dr`, `db`, `dc`, `px`).
- `close_rizin_debug_session`: Terminate debugger session.
