# Sandbox HTTP Actions Integration Guide (Logic Apps Consumption)

This guide documents the HTTP action pattern for Recorded Future and MetaDefender integrations.

## 1) Workflow file

- `workflows/sandbox-http-actions.logicapp.json`

## 2) Required capabilities covered

For **both** systems:

- Submit file via HTTP `POST`
- Parse submit response and extract scan identifier
- Poll analysis result via HTTP `GET` in an `Until` loop
- Parse poll response
- Normalize verdict to one of:
  - `clean`
  - `malicious`
  - `suspicious`
  - `failed`

## 3) Authentication model

Configured with `ManagedServiceIdentity` where possible:

- Recorded Future audience: `https://api.recordedfuture.com`
- MetaDefender audience: `https://api.metadefender.com`

If a provider tenant/app does not support MI token trust, replace with approved enterprise auth method using secure parameters.

## 4) Retry policy

- Submit calls: exponential retry (`count: 4`, `interval: 10s`, max `1m`)
- Poll calls: fixed retry (`count: 2`, `interval: 10s`)

## 5) Polling logic

- `Until` loop for each provider
- Wait step: 30 seconds
- Loop timeout: 3 minutes
- Loop count: 6

## 6) Response parsing and verdict mapping

### Recorded Future normalization

- `malicious` when verdict malicious OR high risk score threshold
- `clean` when verdict clean/benign
- `suspicious` when verdict suspicious or still in progress at timeout
- otherwise `failed`

### MetaDefender normalization

- `malicious` when detected AV count > 0
- `clean` when result indicates no threats detected
- `suspicious` when suspicious indicator or incomplete progress
- otherwise `failed`

## 7) Output contract

Workflow returns:

- `recordedFutureVerdict`
- `metaDefenderVerdict`
- normalized enum list
