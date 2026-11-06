---
name: rea-evidence-verification
description: Forensic evidence management, differential analysis, and reconstruction verification with REA. Manage Evidence IDs, track residual unknowns in session ledgers, diff binary functions/artifacts, and verify clean-room code reconstructions.
---

# Evidence Management & Verification with REA

This skill provides methodologies for maintaining rigorous evidence trails, version diffing, tracking unresolved technical unknowns, and verifying clean-room software reconstructions.

---

## 1. The REA Evidence Model

Every investigation tool in REA outputs an Evidence record. Evidence records provide immutable, auditable proof behind every technical conclusion.

### Anatomy of an Evidence Record
- `evidence_id`: Unique persistent identifier (e.g. `ev_9f82c1b4`).
- `operation`: Name of the tool producing the finding (e.g. `analyze_function`).
- `target_digest`: SHA-256 of the analyzed file or component.
- `normalized_result`: Structured, machine-readable findings.
- `limitations`: Technical constraints or bounded search parameters.
- `_meta.provenance.untrusted_content`: Indicates the payload contains raw data from the analysis target.

---

## 2. Navigating Large Results via Paged Views

When analyzing large binary symbol tables or multi-megabyte JavaScript graphs, tools return an initial summary and retain the full graph under a `parent_evidence_id`.

```json
{
  "source": {
    "evidence_id": "<parent_evidence_id>"
  },
  "view": {
    "kind": "inventory",
    "collection": "nodes",
    "offset": 0,
    "limit": 50
  }
}
```
- Tool: `inspect_analysis_view`
- Query collections (`nodes`, `edges`, `occurrences`, `modules`, `procedures`) iteratively without repeating the original analysis.

### Exporting Forensic Evidence Bundles
To persist an entire investigation's findings across sessions:
```json
{
  "output_path": "/path/to/investigation_evidence.json"
}
```
- Tool: `export_evidence_bundle`
- Serializes all generated Evidence records, relationship graphs, and metadata to disk.

---

## 3. Managing Hypotheses & Unknowns Ledger

In complex investigations, dynamic dispatch, obfuscation, or incomplete artifacts leave technical gaps. Rather than guessing, register them in REA's session ledger:

### Record a Technical Unknown
```json
{
  "topic": "Dynamic Dispatch Target for Function at 0x140005230",
  "missing_authority": "Runtime memory dump or live process attachment",
  "supporting_evidence_ids": ["ev_1024a", "ev_1025b"]
}
```
- Tool: `record_unknown`

### Update Unknown upon Discovery
When subsequent analysis resolves the question:
```json
{
  "unknown_id": "unk_042",
  "status": "resolved",
  "resolution_evidence_id": "ev_2098c",
  "resolution_summary": "Resolved via trace_native_values: target pointer resolves to AuthHandler::ValidateToken"
}
```
- Tool: `update_unknown`

---

## 4. Differential Analysis (Version Comparison)

When auditing security patches, analyzing malware updates, or tracking feature changes between releases:

| Task | Tool | Description |
| :--- | :--- | :--- |
| **Artifact Member Diff** | `compare_artifacts` | Compares archive members, file sizes, and internal checksums. |
| **Function-Level Diff** | `compare_functions` | Compares assembly instructions, control-flow graphs, and pseudocode. |
| **Bundle Membership Diff** | `compare_bundles` | Diffs JavaScript/Electron chunk graphs and dependency manifests. |
| **Managed Member Diff** | `compare_managed_members` | Compares types, fields, and methods in .NET/JVM assemblies. |
| **Behavioral Synthesis** | `find_changed_behavior` | Combines static diffs into high-level behavioral change hypotheses. |

---

## 5. Clean-Room Reconstruction Verification

When reimplementing closed-source libraries or proprietary protocols under clean-room engineering principles:

1. **Build Obligation Ledger**:
   ```json
   {
     "specification": {
       "protocol": "CustomBinaryRPC",
       "version": "2.1",
       "required_endpoints": ["Handshake", "Authenticate", "Heartbeat"]
     }
   }
   ```
   - Tool: `build_reconstruction_obligation_ledger`
   - Defines formal contractual requirements derived strictly from static evidence.

2. **Verify Conformance**:
   ```json
   {
     "ledger_id": "ledger_001",
     "implementation_path": "/path/to/clean_room_code"
   }
   ```
   - Tool: `verify_reconstruction`
   - Validates that the clean-room implementation satisfies all declared obligations without behavioral divergence.
