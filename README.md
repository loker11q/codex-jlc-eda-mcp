# Codex + 嘉立创EDA专业版 MCP Skill

一个面向 Codex 的可复用 Skill，帮助用户连接嘉立创EDA专业版桌面客户端，并通过本地 MCP（模型上下文协议）进行只读检查、原理图/PCB 自动化和故障排查。

## 功能

- 指导配置 Codex 与嘉立创EDA专业版本地 MCP
- 使用 `doctor`、会话列表和只读工具调用验证连接
- 避免错误参数导致的重复启动窗口
- 在实际画图前确认目标工程和关键电气规格
- 通过编辑器 API 操作并验证工程结果，避免盲目改写工程文件

## 安装

### Codex 桌面版

下载或克隆本仓库后，将整个仓库文件夹复制到 Codex Skills 目录：

```text
Windows: %USERPROFILE%\.codex\skills\codex-jlc-eda-mcp
macOS/Linux: ~/.codex/skills/codex-jlc-eda-mcp
```

如本机使用自定义 `CODEX_HOME`，将其下的 `skills` 目录作为安装位置。复制后重启 Codex 或重新加载 Skills。也可以将本仓库放入 Codex 支持的本地技能市场后安装。

### 使用

直接描述需求，或显式调用：

```text
$codex-jlc-eda-mcp 帮我检查 Codex 到嘉立创EDA专业版的 MCP 连接，只读，不要修改工程。
```

连接后进行简单验证：

```text
请用 jlc_eda 列出当前嘉立创EDA会话，并确认是否看到我的工程。只读检查，不修改或保存。
```

`jlc_eda` 应替换为你实际设置的 MCP 服务名。

## 快速配置要点

Codex MCP 服务器使用本机嘉立创EDA专业版可执行文件作为 STDIO 命令。参数必须作为独立项填写：

```text
命令：D:\嘉立创\lceda-pro1\lceda-pro.exe
参数 1：--mcp
参数 2：stdio
```

具体安装路径因电脑而异。不要把 `--mcp stdio` 填作一个参数项；不要用 `open` 来做纯连接测试，因为它会创建新的编辑器窗口。完整步骤见 [`references/setup-and-troubleshooting.md`](references/setup-and-troubleshooting.md)。

## 版本与行为说明

嘉立创 CLI/MCP 要求专业版桌面客户端达到官方文档所述版本。该要求和命令选项会变化，使用前请查看官方文档及本机 `lceda-pro --help`。本 Skill 不会替用户下载、安装或升级软件。

MCP 成功连接不代表电路设计已经验证。对电气设计，仍需明确输入范围、输出电流、器件、散热和板框等关键要求，并按任务进行 ERC/DRC、网表、BOM 和几何检查。

## 文件结构

```text
codex-jlc-eda-mcp/
├── SKILL.md
├── README.md
├── LICENSE
├── agents/
│   └── openai.yaml
└── references/
    └── setup-and-troubleshooting.md
```

## 参考

- [嘉立创EDA专业版命令行客户端](https://prodocs.lceda.cn/cn/api/guide/cli.html)
- [Codex MCP 配置](https://learn.chatgpt.com/docs/extend/mcp?site_variant=chatgpt&translationFallback=zh-Hans)

## License

MIT，详见 [`LICENSE`](LICENSE)。
