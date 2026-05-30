# Trusted A2A Capability Exchange

Experimental trust and settlement layer for agent-to-agent capability exchange.

This repository is not trying to be another plugin store. It explores a lower-level
question: **can an autonomous agent decide whether another agent capability is
trustworthy enough to buy and use?**

The current prototype focuses on verifiable Python capabilities. A seller submits
a Python artifact, a machine-readable manifest, and a replayable eval pack. The
exchange runs the eval pack, produces a scorecard, exposes artifact-free
listings, locks purchase terms through a quote, and releases the artifact only
after mock credit debit and mock escrow creation.

## 中文简介

这是一个面向 Agent 之间能力交易的实验性信任与结算层。

本项目不是普通插件市场，而是在验证一个更底层的问题：**自治 Agent
能否在购买和使用另一个 Agent 能力之前，判断它是否足够可信？**

当前原型聚焦于可验证的 Python 能力。卖家提交 Python artifact、机器可读
manifest 和可复放 eval pack。交易所会运行 eval pack，生成 scorecard，向买家
展示不包含 artifact 的 listing，通过 quote 锁定价格、artifact hash 和
scorecard，并在 mock credit 扣款和 mock escrow 创建后才释放 artifact。

## Documentation / 文档

English:

- [Development Guide](docs/en/development.md)
- [API Manual](docs/en/api.md)
- [Security Notes](docs/en/security.md)
- [Release Notes and Roadmap](docs/en/release.md)
- [Contributing](CONTRIBUTING.md)

中文：

- [开发手册](docs/zh-CN/development.md)
- [API 手册](docs/zh-CN/api.md)
- [安全说明](docs/zh-CN/security.md)
- [发布说明与路线图](docs/zh-CN/release.md)
- [贡献指南](CONTRIBUTING.md)

## Current Prototype

Supported in v0.2:

- Python capability artifacts with a fixed `run(input: dict) -> dict` entrypoint.
- Replayable eval packs submitted at registration time.
- Subprocess verification with timeout limits.
- Machine-readable scorecards.
- Artifact-free discovery responses.
- Quote-locked price, artifact hash, and scorecard snapshots.
- Single-use checkout quotes.
- Mock buyer credit.
- Mock escrow with `held`, `released`, and `disputed` states.
- FastAPI HTTP interface.

Out of scope for this prototype:

- Production sandboxing.
- Real authentication.
- Real payments or clearing.
- Seller balances.
- Human marketplace UI.
- MCP, A2A, WASM, or full agent-core runtime support.
- Reputation beyond the generated scorecard.

## 当前原型范围

v0.2 已支持：

- 固定入口 `run(input: dict) -> dict` 的 Python capability artifact。
- 注册时提交可复放 eval pack。
- 带超时限制的子进程验证。
- 机器可读 scorecard。
- 不暴露 artifact 的 discovery listing。
- 通过 quote 锁定价格、artifact hash 和 scorecard snapshot。
- 一次性 checkout quote。
- Mock buyer credit。
- Mock escrow，状态包括 `held`、`released`、`disputed`。
- FastAPI HTTP 接口。

当前不包含：

- 生产级沙箱。
- 真实认证。
- 真实支付或清结算。
- 卖家余额。
- 人类 marketplace UI。
- MCP、A2A、WASM 或完整 agent-core runtime 支持。
- 超出 scorecard 的声誉系统。

## Quick Start

```bash
git clone https://github.com/aidi1723/agent-exchange-open-source.git
cd agent-exchange-open-source
python3 -m venv .venv
. .venv/bin/activate
pip install '.[dev]'
python -m a2a_exchange
```

Open the interactive API docs:

```text
http://127.0.0.1:8000/docs
```

Run tests:

```bash
.venv/bin/python -m unittest discover -s tests -v
```

## 快速开始

```bash
git clone https://github.com/aidi1723/agent-exchange-open-source.git
cd agent-exchange-open-source
python3 -m venv .venv
. .venv/bin/activate
pip install '.[dev]'
python -m a2a_exchange
```

打开交互式 API 文档：

```text
http://127.0.0.1:8000/docs
```

运行测试：

```bash
.venv/bin/python -m unittest discover -s tests -v
```

## API Summary

| Method | Path                  | Actor  | Purpose                                      |
|--------|-----------------------|--------|----------------------------------------------|
| POST   | `/register`           | seller | Verify and list a Python capability          |
| POST   | `/discover`           | buyer  | Discover verified listings without artifacts |
| POST   | `/quote`              | buyer  | Lock price, artifact hash, and scorecard     |
| POST   | `/checkout`           | buyer  | Debit mock credit and unlock artifact        |
| POST   | `/settle`             | buyer  | Release or dispute mock escrow               |
| GET    | `/balance/{agent_id}` | buyer  | Inspect mock credit                          |
| GET    | `/healthz`            | -      | Liveness plus listing, quote, escrow counts  |

See the full [English API Manual](docs/en/api.md) or [中文 API 手册](docs/zh-CN/api.md).

## Security Boundary

This project is **not safe for public execution of untrusted code**.

The verifier runs submitted Python in a subprocess with timeout controls. That is
useful for local experiments, but it is not a hardened sandbox. Run only locally
or on an isolated network.

安全边界：本项目**不能直接用于公开执行不可信代码**。当前 verifier 只是在子进程
中运行提交的 Python，并做超时控制；这适合本地实验，但不是生产级沙箱。

## License

MIT. See [LICENSE](LICENSE).
