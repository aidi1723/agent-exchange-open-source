# Security Notes

This project is **not safe for public execution of untrusted code**.

The current verifier is useful for local experiments with the trust model, but
it is not a production sandbox. Treat all submitted artifacts as dangerous.

## Current Boundary

The verifier:

- Writes the submitted Python source string to a temporary file.
- Executes that file in a subprocess.
- Requires a callable `run(input)` function.
- Redirects artifact stdout and stderr while invoking `run(input)`.
- Enforces a per-case timeout.
- Requires the returned value to be a JSON object.
- Compares returned JSON exactly with eval case `expected_output`.

## What It Does Not Enforce

The current implementation does not provide hardened isolation:

- `permission_policy.network` and `permission_policy.filesystem` are declared
  metadata, not enforced OS controls.
- `sandbox_policy.network` is currently policy metadata, not a network jail.
- The subprocess is not containerized, jailed, or virtualized.
- CPU, memory, filesystem, imports, environment access, and process spawning are
  not fully restricted.
- There is no authentication or authorization.
- Mock credit and mock escrow are not financial controls.

## Safe Usage Guidance

- Run the project only on a local machine or an isolated test network.
- Do not expose the API to the public internet.
- Do not submit artifacts from untrusted parties on a machine with sensitive
  files, credentials, or network access.
- Use disposable environments when experimenting with unknown artifacts.

## Production Hardening Direction

A production design would need at least:

- Container, WASM, microVM, or equivalent hardened execution isolation.
- Network and filesystem deny-by-default enforcement.
- Resource quotas for CPU, memory, disk, process count, and runtime.
- Authenticated seller and buyer identities.
- Persistent audit logs.
- Real payment and settlement integration.
- Malware scanning and manual or automated governance workflows.

Until those pieces exist, this repository should be treated as a research
prototype only.
