# ContextModeWithProxy

本仓库是 [mksglu/context-mode](https://github.com/mksglu/context-mode)
v1.0.151 的修改版。

上游项目使用 Elastic License 2.0。本仓库保留同样的许可证和上游版权声明。
请查看 [LICENSE](./LICENSE)。

英文主 README: [README.md](./README.md)

上游原始 README: [README.upstream.md](./README.upstream.md)

## 修改内容

这个 fork 面向一种常见场景：context-mode 运行在容器里，而外网访问需要通过本地
Clash/mihomo 代理。

主要修改如下：

- `start.mjs`
  - 在 context-mode MCP server 启动时，如果没有代理环境变量，就注入默认值。
  - 默认值：
    - `HTTP_PROXY=http://127.0.0.1:7890`
    - `HTTPS_PROXY=http://127.0.0.1:7890`
    - `ALL_PROXY=socks5h://127.0.0.1:7891`
    - `NO_PROXY=localhost,127.0.0.1,::1,10.0.0.0/8,192.168.0.0/16,172.16.0.0/12`

- `.codex-plugin/mcp.json`
  - 给 Codex MCP server manifest 添加同样的代理环境变量。

- `src/executor.ts` 和 `server.bundle.mjs`
  - 给 `ctx_execute`、`ctx_execute_file`、`ctx_batch_execute` 等 sandbox
    子进程添加代理环境变量兜底。

- `src/server.ts` 和 `server.bundle.mjs`
  - 关闭上游 `ctx_fetch_and_index` 中删除 proxy 环境变量的逻辑。
  - 给 `ctx_fetch_and_index` 增加代理感知抓取路径：如果检测到代理环境变量，
    生成的抓取脚本会使用 `curl`，因为 `curl` 会读取
    `HTTP_PROXY`、`HTTPS_PROXY`、`ALL_PROXY`，随后仍然走 context-mode 原有的
    markdown 转换和索引存储流程。

更多细节见 [MODIFICATIONS.md](./MODIFICATIONS.md)。

## 为什么需要这个 fork

在部分容器环境里会出现以下情况：

- 交互式 shell 里已经有 `.bashrc` 设置的代理变量；
- Codex 或 MCP 子进程没有继承这些变量；
- Node 22 内置 `fetch` 默认不会读取 `HTTP_PROXY` / `HTTPS_PROXY` /
  `ALL_PROXY`；
- 某些网站直连失败，但通过 Clash 可以访问。

这个 fork 的目标就是让 context-mode 在这种环境下默认可以走代理。

## 安全说明

这个 fork 有意改变了上游 `ctx_fetch_and_index` 的一部分安全行为。

上游会在 fetch 子进程里删除 proxy 环境变量，用来避免代理绕过 DNS rebinding /
SSRF 检查。本 fork 保留 proxy 变量，并在存在 proxy 变量时使用 `curl` 抓取。
这对目标代理环境是必要的，但安全限制比上游版本更弱。

如果你需要抓取不可信 URL，请先确认你接受这个安全取舍。

## 配置

默认代理端口按常见 Clash/mihomo 配置设置：

```bash
HTTP_PROXY=http://127.0.0.1:7890
HTTPS_PROXY=http://127.0.0.1:7890
ALL_PROXY=socks5h://127.0.0.1:7891
```

如果你的代理端口不同，请在启动 Codex/context-mode 前设置对应环境变量。
已有环境变量不会被覆盖；默认值只在变量缺失时生效。

## 已验证

修改版已验证：

- `ctx_execute` sandbox 内可以看到 proxy 环境变量；
- `ctx_execute` 可以访问 `https://news.ycombinator.com/`；
- `ctx_fetch_and_index` 可以通过代理感知路径抓取并索引 Hacker News；
- `ctx_search` 可以检索刚刚索引的 Hacker News 内容；
- `start.mjs`、`server.bundle.mjs`、`.codex-plugin/mcp.json` 语法/JSON 检查通过。

## 许可证

本项目是 context-mode 的衍生作品，继续使用 Elastic License 2.0。
请查看 [LICENSE](./LICENSE)。

