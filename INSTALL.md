# 安装与运行

本仓库是 **Python 项目**（`pyproject.toml` + hatchling），不是 npm 包 ——
npm registry 上没有 `xianyu-mcp`。

## 环境要求

- Python >= 3.10
- 能联网安装 Playwright 浏览器（约 130 MB）

## 安装

```bash
git clone https://github.com/zhangjiabo522/xianyu-mcp.git
cd xianyu-mcp

python3 -m venv .venv
.venv/bin/pip install -e .
# 必须 pin < 2：mcp 2.x 移除了 mcp.server.fastmcp，本项目仍用 v1 API
.venv/bin/pip install "mcp>=1.0,<2"

.venv/bin/playwright install chromium

cp .env.example .env    # 按需修改
```

## 运行

```bash
.venv/bin/xianyu-mcp                    # stdio 方式运行 MCP 服务
# 等价于：.venv/bin/python src/xianyu_mcp/server.py

.venv/bin/python scripts/login.py       # 首次扫码登录，Cookie 落到 ./cookies
.venv/bin/python scripts/check_status.py
```

`scripts/login.py` 需要图形界面扫码，纯无头环境无法完成登录。

## MCP 客户端注册

```json
{
  "mcpServers": {
    "xianyu": {
      "command": "/path/to/xianyu-mcp/.venv/bin/xianyu-mcp",
      "args": [],
      "cwd": "/path/to/xianyu-mcp",
      "env": { "XIANYU_HEADLESS": "false" }
    }
  }
}
```

## 相对上游的改动（可运行性修复）

1. **入口未真正启动 MCP 服务**：`src/xianyu_mcp/server.py` 的 `main_async()`
   原来只 `await stop_event.wait()`，从未调用 transport，stdio 客户端连不上。
   改为 `await mcp.run_stdio_async()`。
2. **依赖上限**：`mcp>=1.0.0` 无上限，pip 会装 2.x 导致 `ImportError`
   （`mcp.server.fastmcp` 已改名为 `MCPServer`）。改为 `mcp>=1.0.0,<2`。
3. **`.env.example` 前缀拼错**：原来写的是 `XIANIU_`，`config.py` 读的是
   `XIANYU_`，且键名重复成 `XIANYU_XIANIU_URL`；已统一为 `XIANYU_XIANYU_URL`
   并指向 `goofish.com`。

## 安装验证（实际跑过）

- `initialize` 握手 OK，serverInfo = `xianyu`
- `tools/list` 返回 18 个工具：hello、check_server_status、search_items、
  publish_item、list_my_items、edit_item、delete_item、get_conversations、
  get_unread_messages、get_sendable_conversations、get_messages、send_message、
  get_item_analytics、get_sales_analytics、get_traffic_analytics、
  get_top_items、analyze_competitors、generate_report
- `tools/call hello` 与 `check_server_status` 正常返回
- Playwright chromium 可无头启动

## 已知不完整的功能

`publish_item` / `edit_item` / `delete_item`（发布与商品管理）、
`get_sales_analytics` / `get_traffic_analytics` / `get_top_items` 在上游仍是
TODO 或半成品；实际可用的是搜索、会话与消息、单品统计与竞品分析。
