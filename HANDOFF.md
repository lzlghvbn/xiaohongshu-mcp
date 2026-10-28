# 交接记录

## 2026-10-28：同步上游 + 做成后台服务

### 合并上游
- 上游 `xpzouying/xiaohongshu-mcp`，已加 remote `upstream`（只 fetch，不推送）。
- 在分支 `fix/search-feeds-wait-stable-timeout` 合并 `upstream/main`，提交 `14c5d4c`，**尚未推送、尚未合入本地 main**。
- 上游新增：发布前关引导卡/建创作者中心会话（#853、#855）、详情页改等关键容器（#814）、搜索筛选跳过默认值与面板内指针路径（#816、#858）。上游最新语义化 tag 为 `v2.5.5`。
- 唯一冲突在 `xiaohongshu/feed_detail.go`：取上游 `waitFeedPageReady`（纯 go-rod、8s 上限），弃用本地的 `noteDetailMap` JS 等待（符合"能用 go-rod 就不注入 JS"）。
- `search.go` 保留本地 `MustWaitDOMStable` 修复，上游没动那一行。
- `go build / vet / test ./...` 全部通过。**未实测** `search_feeds`、`get_feed_detail`（会对账号发请求，测时按 ≥8s 间隔）。

### 登录
- 旧 cookies 已过期，用 `COOKIES_PATH=<仓库>/cookies.json go run ./cmd/login` 重新扫码，seed 沿用 659850124。
- 已验证与 Chrome 里的小红书网页**可同时登录**，Chrome 未掉线；`check_login_status` 返回已登录。
- 不复用 Chrome 的 cookie：会话绑设备指纹，且要解密 Keychain，风险大。

### 后台服务（launchd）
- plist：`~/Library/LaunchAgents/com.lizhilin.xiaohongshu-mcp.plist`，登录后自启、崩溃自动拉起。
- 二进制与更新脚本：`~/Library/Application Support/xiaohongshu-mcp/{xhs-mcp,update.sh}`；改代码后运行 `update.sh`（重新编译并 `launchctl kickstart -k`）。
- 只监听 `127.0.0.1:18060`；日志 `~/Library/Logs/xiaohongshu-mcp.log`。
- 停用：`launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/com.lizhilin.xiaohongshu-mcp.plist`。
- 服务必须跑在本机：cookies/指纹/浏览器/住宅 IP 都在本机，不能放 Cloudflare Workers。

### 客户端配置
- Claude Code CLI：`claude mcp add --transport http --scope user xiaohongshu http://127.0.0.1:18060/mcp`（写在 `~/.claude.json`）。
- Claude Desktop：`claude_desktop_config.json` 的 `mcpServers.xiaohongshu` 用 `/opt/homebrew/bin/npx -y mcp-remote http://127.0.0.1:18060/mcp`；改前备份为同目录 `.bak-20261028`。
- 两个客户端共用一个服务，共用互斥锁和 8s 节流。

### 清理
- 同级旧目录 `~/Workspace/xiaohongshu`（9 月的发布版二进制 + 过期 cookies）已移入废纸篓。

### 待办
- review 合并提交，再决定推送与合入 main（需用户同意后才推）。
- 实测 `search_feeds` / `get_feed_detail`。
- 确认 Claude Desktop 里 xiaohongshu 能出现（首次 `npx` 需下载 `mcp-remote`）。
