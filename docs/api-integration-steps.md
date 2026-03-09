# API Integration Steps - Encrypted Attachment Triage

## 1) Deploy Logic App (Consumption)

Deploy `workflows/soc-email-malware-triage.logicapp.json` as a Consumption workflow.

## 2) Authentication and access model

Use Logic App **system-assigned Managed Identity** for HTTP APIs:

- Defender Quarantine API (`https://api.security.microsoft.com`)
- Sentinel Incident API (`https://management.azure.com/`)
- Log Analytics Logs Ingestion (`https://monitor.azure.com/`)
- Sandbox API and decryption API (when MI trust is supported)

> Note: Exchange Online "When new email arrives" in Consumption uses Office 365 connector auth; scope that connector identity to `cyber@abc.com`.

## 3) Trigger and filter configuration

- Trigger: Office 365 `When a new email arrives`.
- Mailbox target: `cyber@abc.com`.
- Subject filter: `[WARNING: MESSAGE ENCRYPTED]`.

## 4) Core processing sequence

1. Parse password from body via regex: `Password[:= ]\s*(\S+)`.
2. Parse Message-ID header for correlation.
3. Query Defender quarantine by Message-ID.
4. Download quarantined attachments.
5. For each attachment:
   - decrypt via decryption API with extracted password
   - submit decrypted payload to sandbox API
   - capture scan ID
   - poll status every 30 seconds
   - stop after 3 minutes
   - retry one additional polling cycle if no verdict
6. Aggregate attachment verdicts.

## 5) Decision logic

- If **any** attachment verdict is `malicious`:
  - create Sentinel incident via Incident API.
- If **all** attachment verdicts are `clean`:
  - release message from quarantine via Defender API.
- If verdict is `suspicious` or scan failed/time-out:
  - send SOC notification email for manual investigation.

## 6) Polling settings

- Interval: `30` seconds
- Max duration per cycle: `3` minutes
- Retries if no verdict: `1` full additional polling cycle

## 7) Logging to Log Analytics

Use Logs Ingestion API custom stream/table and write structured records with:

- `Sender`
- `Recipient`
- `AttachmentName`
- `SandboxVerdict`
- `Timestamp`
- `ActionTaken`

Include tracking identifiers and result metadata for correlation.

## 8) Error handling

- Per-action retry policies for outbound HTTP actions.
- Global failure scope:
  - notify SOC,
  - terminate run with explicit failure code.
