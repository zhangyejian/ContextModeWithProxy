# ContextModeWithProxy

This repository is a modified distribution of
[mksglu/context-mode](https://github.com/mksglu/context-mode) v1.0.151.

The upstream project is licensed under the Elastic License 2.0. This repository
keeps the same license and preserves the upstream copyright notice. See
[LICENSE](./LICENSE).

Chinese documentation: [README.zh-CN.md](./README.zh-CN.md)

Original upstream README: [README.upstream.md](./README.upstream.md)

## What Changed

This fork is intended for environments where context-mode runs inside a
container and outbound web access must go through a local Clash/mihomo proxy.

The relevant changes are:

- `start.mjs`
  - Adds default proxy environment variables when they are not already set.
  - Defaults:
    - `HTTP_PROXY=http://127.0.0.1:7890`
    - `HTTPS_PROXY=http://127.0.0.1:7890`
    - `ALL_PROXY=socks5h://127.0.0.1:7891`
    - `NO_PROXY=localhost,127.0.0.1,::1,10.0.0.0/8,192.168.0.0/16,172.16.0.0/12`

- `.codex-plugin/mcp.json`
  - Adds the same proxy variables to the Codex MCP server manifest so the
    context-mode MCP process receives them when the host respects manifest env.

- `src/executor.ts` and `server.bundle.mjs`
  - Adds proxy defaults to sandbox subprocess environments used by tools such as
    `ctx_execute`, `ctx_execute_file`, and `ctx_batch_execute`.

- `src/server.ts` and `server.bundle.mjs`
  - Disables the upstream proxy-env stripping in `ctx_fetch_and_index`.
  - Adds a proxy-aware fetch path for `ctx_fetch_and_index`: when proxy
    variables are present, the generated fetch script uses `curl`, which honors
    `HTTP_PROXY`, `HTTPS_PROXY`, and `ALL_PROXY`, then continues through the
    normal context-mode markdown conversion and indexing path.

See [MODIFICATIONS.md](./MODIFICATIONS.md) for implementation details.

## Why This Exists

In some containerized setups:

- the interactive shell has proxy variables from `.bashrc`;
- the Codex or MCP child process does not inherit them;
- Node 22's built-in `fetch` does not automatically use
  `HTTP_PROXY` / `HTTPS_PROXY` / `ALL_PROXY`;
- direct outbound access to some sites may fail, while access through Clash
  works.

This fork makes context-mode proxy-aware by default for that environment.

## Security Notes

This fork intentionally changes part of upstream's `ctx_fetch_and_index`
security behavior.

Upstream removed proxy environment variables from the fetch subprocess to avoid
proxy-based bypasses of its DNS rebinding / SSRF checks. This fork keeps proxy
variables and uses `curl` when proxy variables are present. That is necessary
for the target proxy-based environment, but it is less restrictive than the
upstream design.

Do not use this fork for untrusted URL fetching unless you accept that tradeoff.

## Configuration

The default proxy endpoints assume a common Clash/mihomo setup:

```bash
HTTP_PROXY=http://127.0.0.1:7890
HTTPS_PROXY=http://127.0.0.1:7890
ALL_PROXY=socks5h://127.0.0.1:7891
```

If your proxy uses different ports, set those environment variables before
starting Codex/context-mode. Existing values are preserved; the defaults are only
used when the variables are missing.

## Verification Performed

The modified version was tested with:

- `ctx_execute` reading proxy variables inside the sandbox;
- `ctx_execute` successfully fetching `https://news.ycombinator.com/`;
- `ctx_fetch_and_index` successfully fetching and indexing Hacker News through
  the proxy-aware path;
- `ctx_search` retrieving content from the indexed Hacker News source;
- syntax checks for `start.mjs`, `server.bundle.mjs`, and `.codex-plugin/mcp.json`.

## License

This is a derivative work of context-mode and remains under the Elastic License
2.0. See [LICENSE](./LICENSE).

