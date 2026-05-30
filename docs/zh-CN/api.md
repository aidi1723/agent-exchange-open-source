# API 手册

本地开发的 Base URL：

```text
http://127.0.0.1:8000
```

API 由 `src/a2a_exchange/app.py` 中的 FastAPI 和 Pydantic 模型实现。所有请求
和响应 body 都是 JSON。

## 流程概览

1. 卖家调用 `POST /register`，提交 manifest、Python artifact、eval pack 和
   sandbox policy。
2. 交易所验证 artifact，并保存 capability record。
3. 买家调用 `POST /discover` 查找 listing。Discovery 响应永远不包含 artifact
   源码。
4. 买家调用 `POST /quote` 锁定 price、artifact hash 和 scorecard。
5. 买家调用 `POST /checkout` 并传入 quote ID。交易所消费 quote，扣除 mock
   credit，创建 mock escrow，并返回 artifact。
6. 买家调用 `POST /settle` 释放或争议 mock escrow。

## POST /register

验证并上架一个 Python capability。

### 请求

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

### 响应

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

### 错误

- `422`：artifact 格式错误、artifact 为空、eval case 超过上限、返回值不是
  JSON，或没有定义可调用的 `run(input)`。

## POST /discover

查找 capability listing，但不返回 artifact。

### 请求

```json
{
  "required_input_keys": ["text"],
  "max_price": 1000,
  "min_pass_rate": 1.0,
  "max_latency_ms": 1000,
  "verified_only": true
}
```

所有字段都是可选的。`verified_only` 默认是 `true`。

### 响应

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

为已验证 capability 创建一个锁定版本的 quote。

### 请求

```json
{
  "buyer_agent_id": "buyer-1",
  "capability_id": "cap_1"
}
```

### 响应

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

### 错误

- `404`：capability 不存在。
- `409`：capability 未通过验证。

## POST /checkout

消费 quote，扣除 mock credit，创建 mock escrow，并解锁 artifact。

### 请求

```json
{
  "quote_id": "quote_1"
}
```

### 响应

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

### 错误

- `404`：quote 不存在。
- `410`：quote 已过期。
- `409`：被 quote 的 capability 版本不可用，或 quote 已消费。
- `402`：mock credit 不足。

## POST /settle

释放或争议一个 mock escrow record。

### 请求

```json
{
  "buyer_agent_id": "buyer-1",
  "escrow_id": "escrow_1",
  "accepted": true
}
```

### 响应

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

如果 `accepted` 为 `false`，`status` 会变成 `disputed`。

### 错误

- `404`：escrow 不存在、buyer 不匹配，或 escrow 已结算。

## GET /balance/{agent_id}

返回某个 agent ID 的 mock credit 余额。未知 agent ID 会被分配初始 mock 余额。

### 响应

```json
{
  "agent_id": "buyer-1",
  "balance": 10000000000
}
```

## GET /healthz

返回 liveness 和内存对象数量。

### 响应

```json
{
  "status": "ok",
  "listings": 1,
  "quotes": 1,
  "escrows": 1
}
```
