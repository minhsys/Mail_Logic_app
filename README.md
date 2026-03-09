# Azure Logic Apps (Consumption) - Encrypted Attachment Malware Automation

This repo contains a production SOC automation workflow for password-protected email attachments quarantined by Exchange Online/Defender.

## Workflow included

- `workflows/soc-email-malware-triage.logicapp.json`
  - Trigger: Exchange Online **When new email arrives** for `cyber@abc.com`.
  - Filter: subject contains `[WARNING: MESSAGE ENCRYPTED]`.
  - Extract password using regex `Password[:= ]\s*(\S+)`.
  - Extract Message-ID and locate original quarantined message in Defender.
  - Download quarantined attachments.
  - For each attachment:
    - decrypt with provided password (decryption API)
    - submit to sandbox API
    - poll every 30 seconds up to 3 minutes
    - retry polling cycle once when verdict is not completed
  - Decision:
    - any malicious => create Sentinel incident
    - all clean => release email from quarantine
    - suspicious/scan failure => notify SOC
  - Logging to Log Analytics with structured fields:
    - sender
    - recipient
    - attachment name
    - sandbox verdict
    - timestamp
    - action taken

## Security and resiliency

- Managed Identity for Defender, Sentinel API, Log Analytics ingestion, and internal APIs where supported.
- Retry policies applied on outbound HTTP calls.
- Global failure handler notifies SOC and terminates with explicit failure code.
