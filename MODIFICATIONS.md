# Modification Notice

This repository contains a modified copy of `mksglu/context-mode` v1.0.151.

Upstream:

- Repository: https://github.com/mksglu/context-mode
- License: Elastic License 2.0
- Original copyright: Copyright 2026 Mert Koseoglu

## Modified Files

The proxy-related modifications are intentionally limited to:

- `.codex-plugin/mcp.json`
- `start.mjs`
- `src/executor.ts`
- `src/server.ts`
- `server.bundle.mjs`

The upstream `README.md` has been preserved as `README.upstream.md`.

## Behavioral Changes

1. Proxy variables are injected by default when absent:

   ```text
   HTTP_PROXY=http://127.0.0.1:7890
   HTTPS_PROXY=http://127.0.0.1:7890
   ALL_PROXY=socks5h://127.0.0.1:7891
   NO_PROXY=localhost,127.0.0.1,::1,10.0.0.0/8,192.168.0.0/16,172.16.0.0/12
   ```

2. Sandbox execution tools inherit or receive proxy defaults:

   - `ctx_execute`
   - `ctx_execute_file`
   - `ctx_batch_execute`

3. `ctx_fetch_and_index` no longer strips proxy environment variables.

4. `ctx_fetch_and_index` uses a `curl`-based proxy-aware fetch path when proxy
   variables are present. The fetched body still continues through context-mode's
   normal content conversion and indexing pipeline.

## Security Tradeoff

The upstream implementation stripped proxy variables in the fetch subprocess as
part of its DNS rebinding / SSRF defense. This fork changes that behavior to
support environments where all outbound traffic must go through Clash/mihomo.

This is intentional, but less restrictive than upstream. Users should avoid
fetching untrusted URLs unless they accept that tradeoff.

