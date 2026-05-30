# Development Guide

This guide explains how to run, test, and extend the Trusted A2A Capability
Exchange prototype.

## Requirements

- Python 3.9 or newer.
- `pip` with PEP 517 build support.
- A local machine or isolated network. Do not expose this prototype to public
  untrusted code execution.

## Setup

```bash
git clone https://github.com/aidi1723/agent-exchange-open-source.git
cd agent-exchange-open-source
python3 -m venv .venv
. .venv/bin/activate
pip install '.[dev]'
```

If editable install is not supported by an older `pip`, use the non-editable
form shown above or upgrade pip inside the virtual environment.

## Run the API

```bash
python -m a2a_exchange
```

The application starts a Uvicorn server on:

```text
http://127.0.0.1:8000
```

FastAPI interactive documentation is available at:

```text
http://127.0.0.1:8000/docs
```

## Run Tests

```bash
.venv/bin/python -m unittest discover -s tests -v
```

The current test suite covers registration, failed verification, artifact-free
discovery, quote locking, checkout, credit debit, escrow settlement, and quote
stability after later registrations.

## Project Layout

```text
src/a2a_exchange/
  manifest.py    Pydantic models for manifests, policies, scorecards, listings
  eval_pack.py   Replayable eval case and eval pack models
  verifier.py    Subprocess-based Python artifact verification
  registry.py    Thread-safe in-memory capability registry
  discovery.py   Artifact-free discovery filters
  quote.py       Version-locked, single-use quote book
  escrow.py      Mock escrow state machine
  credit.py      Mock static buyer credit guard
  app.py         FastAPI HTTP surface and app factory
  __main__.py    Uvicorn launcher
tests/
  test_exchange.py   End-to-end trusted exchange flow tests
docs/
  en/          English manuals
  zh-CN/       Chinese manuals
```

## Runtime Model

The application uses in-memory state by default:

- `CapabilityRegistry` stores submitted capability records.
- `CapabilityDiscovery` returns listing metadata without artifacts.
- `QuoteBook` creates short-lived, single-use quotes that snapshot price,
  artifact hash, and scorecard.
- `MockCreditGuard` gives each buyer a static mock token balance.
- `EscrowBook` records mock settlement state.

Restarting the process clears all registry, quote, balance, and escrow state.

## Capability Contract

Artifacts are Python source strings. A valid artifact must define:

```python
def run(input):
    return {"result": "json-compatible object"}
```

The verifier expects `run(input)` to return a JSON object, represented in Python
as a `dict`. Eval case outputs are compared by exact equality with
`expected_output`.

## Development Guidelines

- Keep public behavior covered by tests in `tests/test_exchange.py` or focused
  new test files.
- Keep API schemas in `src/a2a_exchange/app.py` synchronized with
  `docs/en/api.md` and `docs/zh-CN/api.md`.
- Preserve the current security warning unless a real hardened sandbox is added.
- Avoid adding persistent storage, authentication, or real payments without a
  dedicated design update.

## Useful Commands

```bash
# Run all tests
.venv/bin/python -m unittest discover -s tests -v

# Start the API
.venv/bin/python -m a2a_exchange

# Inspect current git state
git status --short --branch
```
