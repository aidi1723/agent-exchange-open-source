# Release Notes and Roadmap

## Current Version

Version: `0.2.0`

Repository:

```text
https://github.com/aidi1723/agent-exchange-open-source
```

License: MIT

Status: prototype.

## v0.2 Capabilities

- Python capability registration through `POST /register`.
- Replayable eval pack verification.
- Machine-readable scorecards.
- Artifact-free discovery through `POST /discover`.
- Quote-locked checkout terms through `POST /quote`.
- Single-use quote checkout through `POST /checkout`.
- Mock credit debit.
- Mock escrow creation and settlement.
- FastAPI app factory for tests and embedding.

## Known Limitations

- In-memory state only.
- No authentication.
- No real payment flow.
- No production sandbox.
- No seller balances.
- No human marketplace UI.
- No persistent audit log.
- No reputation model beyond scorecards.

## Roadmap

Near-term:

- Stronger execution isolation with container, WASM, or microVM verification.
- Persistent registry, quote, escrow, and audit storage.
- API key or token authentication.
- Human governance console for publishing, review, budgets, and risk policy.

Research direction:

- Capability provenance and identity.
- Reputation from verified usage outcomes.
- Escrow release tied to post-purchase acceptance tests.
- MCP/A2A adapter support.
- Cost and permission proofs that buyer agents can evaluate automatically.

## Release Checklist

Before publishing a release:

```bash
.venv/bin/python -m unittest discover -s tests -v
git status --short --branch
```

Confirm:

- Tests pass.
- README and docs match current API behavior.
- Security notes still describe the real boundary.
- Version in `pyproject.toml` matches the intended release.
