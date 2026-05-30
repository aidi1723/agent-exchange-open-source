# 发布说明与路线图

## 当前版本

版本：`0.2.0`

仓库：

```text
https://github.com/aidi1723/agent-exchange-open-source
```

许可证：MIT

状态：原型。

## v0.2 能力

- 通过 `POST /register` 注册 Python capability。
- 可复放 eval pack 验证。
- 机器可读 scorecard。
- 通过 `POST /discover` 做不暴露 artifact 的 discovery。
- 通过 `POST /quote` 锁定 checkout 条件。
- 通过 `POST /checkout` 消费一次性 quote。
- Mock credit 扣款。
- Mock escrow 创建和结算。
- 用于测试和嵌入的 FastAPI app factory。

## 已知限制

- 只有内存状态。
- 没有认证。
- 没有真实支付流程。
- 没有生产级沙箱。
- 没有卖家余额。
- 没有人类 marketplace UI。
- 没有持久审计日志。
- 除 scorecard 外没有声誉模型。

## 路线图

近期方向：

- 使用 container、WASM 或 microVM 强化执行隔离。
- 持久化 registry、quote、escrow 和 audit 存储。
- API key 或 token 认证。
- 面向发布、审核、预算和风险策略的人类治理控制台。

研究方向：

- Capability provenance 和 identity。
- 基于已验证使用结果的 reputation。
- 将 escrow release 绑定到购买后的 acceptance tests。
- MCP/A2A adapter 支持。
- 买家 Agent 可自动评估的成本和权限证明。

## 发布检查清单

发布前运行：

```bash
.venv/bin/python -m unittest discover -s tests -v
git status --short --branch
```

确认：

- 测试通过。
- README 和 docs 与当前 API 行为一致。
- 安全说明仍然准确描述真实边界。
- `pyproject.toml` 中的版本号符合预期发布版本。
