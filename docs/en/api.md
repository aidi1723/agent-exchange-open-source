# API Manual

Base URL for local development:

```text
http://127.0.0.1:8000
```

The API is implemented in `src/a2a_exchange/app.py` with FastAPI and Pydantic
models. All request and response bodies are JSON.

## Flow Overview

1. Seller calls `POST /register` with a manifest, Python artifact, eval pack,
   and sandbox policy.
2. The exchange verifies the artifact and stores a capability record.
3. Buyer calls `POST /discover` to find listings. Discovery responses never
   include the artifact source.
4. Buyer calls `POST /quote` to lock price, artifact hash, and scorecard.
5. Buyer calls `POST /checkout` with the quote ID. The exchange consumes the
   quote, debits mock credit, creates mock escrow, and returns the artifact.
6. Buyer calls `POST /settle` to release or dispute the mock escrow.

## POST /register

Verifies and lists a Python capability.

### Request

```json
{
  "manifest": {
    "name": "uppercase-tool",
    "interface": {
      "input_schema": {
        "type": "object",
        "properties": {
          "text": { "type": "string" }
        },
        "required": ["text"]
      },
      "output_schema": {
        "type": "object",
        "properties": {
          "upper": { "type": "string" }
        },
        "required": ["upper"]
      }
    },
    "price_tokens": 500,
    "permission_policy": {
      "network": false,
      "filesystem": false
    },
    "description": "Converts text to uppercase."
  },
  "artifact": "def run(input):\n    return {\"upper\": input[\"text\"].upper()}",
  "eval_pack": {
    "cases": [
      {
        "name": "hello uppercase",
        "input": { "text": "hello" },
        "expected_output": { "upper": "HELLO" }
      }
    ]
  },
  "sandbox_policy": {
    "network": false,
    "timeout_ms": 1000,
    "max_cases": 5
  }
}
```

### Response

```json
{
  "capability_id": "cap_1",
  "verification_status": "verified",
  "scorecard": {
    "verified": true,
    "pass_rate": 1.0,
    "cases_total": 1,
    "cases_passed": 1,
    "avg_latency_ms": 5,
    "artifact_sha256": "64-character-sha256-hex-string",
    "case_results": [
      {
        "name": "hello uppercase",
        "passed": true,
        "duration_ms": 5,
        "error": ""
      }
    ]
  }
}
```

### Errors

- `422`: malformed artifact, empty artifact, too many eval cases, non-JSON
  output, or missing callable `run(input)`.

## POST /discover

Finds capability listings without returning artifacts.

### Request

```json
{
  "required_input_keys": ["text"],
  "max_price": 1000,
  "min_pass_rate": 1.0,
  "max_latency_ms": 1000,
  "verified_only": true
}
```

All fields are optional. `verified_only` defaults to `true`.

### Response

```json
[
  {
    "capability_id": "cap_1",
    "name": "uppercase-tool",
    "interface": {
      "input_schema": {
        "type": "object",
        "properties": {
          "text": { "type": "string" }
        },
        "required": ["text"]
      },
      "output_schema": {
        "type": "object",
        "properties": {
          "upper": { "type": "string" }
        },
        "required": ["upper"]
      }
    },
    "price_tokens": 500,
    "permission_policy": {
      "network": false,
      "filesystem": false
    },
    "description": "Converts text to uppercase.",
    "verification_status": "verified",
    "scorecard": {
      "verified": true,
      "pass_rate": 1.0,
      "cases_total": 1,
      "cases_passed": 1,
      "avg_latency_ms": 5,
      "artifact_sha256": "64-character-sha256-hex-string",
      "case_results": []
    }
  }
]
```

## POST /quote

Creates a version-locked quote for a verified capability.

### Request

```json
{
  "buyer_agent_id": "buyer-1",
  "capability_id": "cap_1"
}
```

### Response

```json
{
  "quote_id": "quote_1",
  "buyer_agent_id": "buyer-1",
  "capability_id": "cap_1",
  "artifact_sha256": "64-character-sha256-hex-string",
  "price_tokens": 500,
  "scorecard_snapshot": {
    "verified": true,
    "pass_rate": 1.0,
    "cases_total": 1,
    "cases_passed": 1,
    "avg_latency_ms": 5,
    "artifact_sha256": "64-character-sha256-hex-string",
    "case_results": []
  },
  "expires_at": "2026-05-30T14:00:00.000000Z"
}
```

### Errors

- `404`: capability not found.
- `409`: capability is not verified.

## POST /checkout

Consumes a quote, debits mock credit, creates mock escrow, and unlocks the
artifact.

### Request

```json
{
  "quote_id": "quote_1"
}
```

### Response

```json
{
  "status": "unlocked",
  "quote_id": "quote_1",
  "escrow_id": "escrow_1",
  "capability_id": "cap_1",
  "artifact_sha256": "64-character-sha256-hex-string",
  "price_paid": 500,
  "remaining_balance": 9999999500,
  "artifact": "def run(input):\n    return {\"upper\": input[\"text\"].upper()}"
}
```

### Errors

- `404`: quote not found.
- `410`: quote expired.
- `409`: quoted capability version unavailable or quote already consumed.
- `402`: insufficient mock credit.

## POST /settle

Releases or disputes a mock escrow record.

### Request

```json
{
  "buyer_agent_id": "buyer-1",
  "escrow_id": "escrow_1",
  "accepted": true
}
```

### Response

```json
{
  "escrow_id": "escrow_1",
  "quote_id": "quote_1",
  "buyer_agent_id": "buyer-1",
  "capability_id": "cap_1",
  "artifact_sha256": "64-character-sha256-hex-string",
  "amount_tokens": 500,
  "status": "released"
}
```

If `accepted` is `false`, `status` becomes `disputed`.

### Errors

- `404`: escrow not found or buyer mismatch.

## GET /balance/{agent_id}

Returns the mock credit balance for an agent ID. Unknown agent IDs are assigned
the initial mock balance.

### Response

```json
{
  "agent_id": "buyer-1",
  "balance": 10000000000
}
```

## GET /healthz

Returns liveness and in-memory object counts.

### Response

```json
{
  "status": "ok",
  "listings": 1,
  "quotes": 1,
  "escrows": 1
}
```
