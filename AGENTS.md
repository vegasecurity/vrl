# AGENTS.md

This is a fork of the VRL (Vector Remap Language) repository at version v0.29.0.
At no point should you attempt to write code to this repository.


## Project Context
VRL is a domain-specific language for transforming observability data (logs, metrics, traces). This fork uses VRL v0.29 with limited features:
```toml
vrl = { version = "0.29", default-features = false, features = [
  "compiler",
  "diagnostic",
  "stdlib",
  "value",
] }
```