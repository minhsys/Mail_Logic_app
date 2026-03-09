# Azure Logic Apps (Consumption) - Sandbox HTTP Integrations for SOC Automation

This repository contains production-style Azure Logic App (Consumption) definitions for sandbox integration patterns used in email malware automation.

## Workflow artifact

- `workflows/sandbox-http-actions.logicapp.json`
  - Uses **HTTP actions** for:
    - Recorded Future sandbox
      - submit file
      - parse submit response for scan ID
      - poll result using `Until`
      - parse poll response
      - normalize verdict to: `clean | malicious | suspicious | failed`
    - MetaDefender sandbox
      - submit file
      - parse submit response for `data_id`
      - poll result using `Until`
      - parse poll response
      - normalize verdict to: `clean | malicious | suspicious | failed`
  - Uses **Managed Identity** auth audiences where possible.
  - Applies retry policies on outbound HTTP calls.
  - Returns normalized verdicts in a final HTTP response.

## Standard polling configuration

- Poll interval: 30 seconds
- Max duration: 3 minutes
- Polling implemented with `Until` limits (`count` + `timeout`)

## Notes

- Validate target API MI support in your tenant; if third-party APIs do not support Entra-issued tokens, swap auth method via secure configuration.
