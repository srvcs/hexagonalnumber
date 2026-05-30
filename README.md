# srvcs-hexagonalnumber

Sequences microservice for srvcs.cloud: computes the **nth hexagonal number**.

## Concern

`sequences: nth hexagonal number`

The hexagonal numbers are the figurate numbers `H(n) = n * (2n - 1)`:
`1, 6, 15, 28, 45, 66, ...` (for `n = 1, 2, 3, ...`).

## Algorithm

Given `n = value` (an `i64`):

1. compute `m = 2n - 1` locally (index arithmetic);
2. ask `srvcs-multiply` for `result = n * m`.

So `H(5) = 5 * 9 = 45` and `H(1) = 1 * 1 = 1`.

This service is an **orchestrator**: the defining product is delegated to a
dependency service over HTTP. It never calls `srvcs-isnumber` directly —
validation propagates from its dependency (a forwarded `422`).

## Dependencies

| Service          | Purpose                           | URL env var          | Default                 |
| ---------------- | --------------------------------- | -------------------- | ----------------------- |
| `srvcs-multiply` | the defining product `n * (2n-1)` | `SRVCS_MULTIPLY_URL` | `http://127.0.0.1:8092` |

## API

### `GET /` — identity

```json
{
  "service": "srvcs-hexagonalnumber",
  "concern": "sequences: nth hexagonal number",
  "depends_on": ["srvcs-multiply"]
}
```

### `POST /` — evaluate

Request:

```json
{ "value": 5 }
```

Response `200`:

```json
{ "value": 5, "result": 45 }
```

Error responses:

- `422` — a dependency rejected the input (forwarded verbatim).
- `500` — a dependency returned a malformed result (missing integer `result`).
- `503` — a dependency is unavailable (degraded).

## Other endpoints

- `GET /healthz` — liveness
- `GET /readyz` — readiness
- `GET /metrics` — Prometheus metrics
- `GET /openapi.json` — OpenAPI document

## Development

```sh
UPDATE_OPENAPI=1 cargo test --offline --test openapi_snapshot
cargo fmt && cargo fmt --check
cargo clippy --offline --all-targets -- -D warnings
cargo test --offline
```
