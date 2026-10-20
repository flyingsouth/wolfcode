# WolfCode

终端里的 AI 编程助手：以 ReAct 循环驱动大模型读写代码、执行命令、调用 MCP 工具，支持多模型协议接入与多 Agent 并行协作。

## 特性

- **多协议接入**：Anthropic Messages 与 OpenAI（含 vLLM / Ollama 等兼容端点）统一到一套消息抽象，接口层不感知厂商差异。
- **工具与权限**：权限判定多层短路（Plan 白名单 → 安全命令 → 危险命令黑名单 → 沙箱自动放行 → 路径沙箱 → 规则引擎 → 模式矩阵 → 人工确认），复合命令按 `&&` `||` `;` `|` 拆条校验；可叠加 bwrap（Linux）/ seatbelt（macOS）内核级沙箱。
- **上下文管理**：工具结果按聚合预算落盘并留预览，压缩时保留近端原文、对前缀做摘要，压缩后重注入最近读过的文件、已激活 Skill 与工具清单；压缩边界持久化，会话可恢复。
- **MCP 工具装载**：按上下文占用与端点能力选择全量 / `defer_loading` / 统一分发三种模式，尽量保持 `tools[]` 字节不变，避免打断 prompt cache 前缀。
- **多 Agent 协作**：每个子 Agent 拥有独立 git worktree 与路径沙箱，通过 mailbox 与共享任务列表协作，工具集按角色裁剪。
- **长期记忆**：用户 / 项目两级 Markdown 存储，`WOLFCODE.md` 指针索引，召回由 LLM 侧选，空闲时由 consolidation 子 Agent 合并去重。
- **生命周期钩子**：`session_start` / `pre_tool_use` / `post_tool_use` / `compact` 等十余个事件，支持 command / prompt / http / agent 四类动作。

## 快速开始

要求 Python ≥ 3.11。

```bash
pip install -e .

# 复制配置样例并填入自己的 API Key
cp .wolfcode/config.yaml.example .wolfcode/config.yaml

# 启动
wolfcode
```

配置示例（`.wolfcode/config.yaml`）：

```yaml
providers:
  - name: anthropic-official
    protocol: anthropic
    base_url: https://api.anthropic.com
    api_key: "your-api-key-here"
    model: claude-sonnet-4-20250514

permission_mode: default
```

也支持 OpenAI 协议端点与 MCP server，详见 `.wolfcode/config.yaml.example`。

## 目录结构

```
wolfcode/
  __main__.py       CLI 入口
  agent.py          Agent 主循环（ReAct、工具调度、压缩触发）
  app.py            终端 UI
  client.py         Anthropic / OpenAI 协议客户端
  context/          上下文预算与压缩
  tools/            内置工具（读写文件、Bash、任务、Agent 等）
  mcp/              MCP 客户端、工具包装与装载策略
  permissions/      权限判定、规则引擎、路径沙箱
  sandbox/          bwrap / seatbelt 封装
  memory/           长期记忆、召回与合并
  teams/            多 Agent 协作（mailbox、共享任务、spawn）
  worktree/         git worktree 隔离与清理
  hooks/            生命周期钩子
tests/              单元测试与集成测试
```

## 测试

```bash
pip install pytest pytest-asyncio
pytest
```

## 说明

运行时数据（会话记录、worktree、配置）默认落在 `.wolfcode/` 目录下；该目录已在 `.gitignore` 中忽略，不会提交。
