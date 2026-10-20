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

## 完整连接流程

连接由三部分组成：嘉立创EDA专业版桌面客户端、本机 CLI/MCP 服务、Codex。CLI/MCP 服务通过本地桥接连接到正在运行的 EDA 编辑器。

### 1. 确认桌面客户端和版本

安装并启动嘉立创EDA专业版**桌面客户端**，打开要测试的工程。浏览器网页版不提供这里使用的本地会话桥接。CLI/MCP 能力与客户端版本有关；请查看[嘉立创官方 CLI 文档](https://prodocs.lceda.cn/cn/api/guide/cli.html)及本机帮助，不要只根据本文中的版本号判断。

Windows 下用 PowerShell 检查程序路径和命令：

```powershell
$app = 'D:\嘉立创\lceda-pro1\lceda-pro.exe'
& $app --help | Out-String
& $app doctor | Out-String
```

将 `$app` 路径换成你电脑上实际的 `lceda-pro.exe` 路径。编辑器运行时，`doctor` 预期返回 JSON，其中应有 `"ok":true` 和 `"connected":true`；`versionMatch` 应为 `true`。如果 PowerShell 没显示输出，可保留 `| Out-String` 再试，并查看该版本 `--help` 的 PowerShell 提示。

### 2. 在 Codex 添加嘉立创 MCP

打开 Codex 桌面版的 **Settings（设置）→ MCP servers → Add server（添加服务器）**，按下面填写：

```text
名称：jlc_eda                 # 可自定义；后续测试提示词要用同一个名称
类型：STDIO
启动命令：D:\嘉立创\lceda-pro1\lceda-pro.exe
参数项 1：--mcp
参数项 2：stdio
工作目录：D:\嘉立创\lceda-pro1
环境变量：留空
环境变量传递：留空
```

把示例路径替换为真实安装路径。**两个参数要分开填写**：Codex 会把每一项作为一个命令行参数传递。不要把 `--mcp stdio` 填在同一参数项里。工作目录应是存在的目录；通常可填 EDA 程序所在目录。

保存后按 Codex 提示重启或重新加载 MCP。若设置变更后当前聊天仍看不到新工具，重新打开 Codex 或新建聊天，再检查 `/mcp` 中是否列出 `jlc_eda`。

### 3. 做只读连接测试

先检查嘉立创命令行和当前会话：

```powershell
& $app doctor | Out-String
& $app session list | Out-String
```

`session list` 是只读查询，不会新开编辑器窗口。纯连接测试不要执行 `lceda-pro open`，因为 `open` 会创建新的编辑器会话。

然后在 Codex 新聊天中发送：

```text
请使用 jlc_eda 列出当前嘉立创EDA会话，并判断是否看到工程“我的工程名”。
这是只读测试，不要新开工程，不要修改或保存任何文件。
```

将 `jlc_eda` 和工程名替换为实际值。测试成功需要看到 MCP 工具实际返回的会话/工程信息；仅显示“服务器已启用”不代表工具调用已成功。确认目标工程和会话后，再发出绘图或编辑请求。

### 4. 使用 Codex 绘制工程

提出设计任务时，写清楚要新建还是修改哪个工程、需要原理图还是 PCB、输入/输出规格、电流、接口、板框等要求。比如“5V 转 3.3V”还需要输出电流和稳压方案等信息；关键参数缺失时先补充，或明确约定假设。

让 Codex 通过嘉立创 MCP/API 创建和检查设计。要求它核对器件型号、引脚、封装、网络连接，并报告 ERC/DRC 结果和未解决问题。不要仅凭“画完了”或一次 API 调用成功就认定电路可投产；制造文件只在明确需要时导出。

### 5. 故障排查

- **Codex 找不到工具**：确认 MCP 服务已启用，重载 Codex MCP；在 `/mcp` 检查服务名；确认 EDA 桌面客户端正在运行，再检查 `doctor`。
- **不断弹出嘉立创窗口或出现白屏闪动**：先禁用 Codex 中的 `jlc_eda` 并重启 Codex，阻止重复启动；确认命令指向 CLI 可执行文件，参数为分开的 `--mcp` 与 `stdio`，工作目录有效。修正后只重新启用一次并做只读调用。
- **禁用 MCP 后仍白屏/闪动**：这说明症状仍出现在 MCP 服务未运行时，应进一步检查嘉立创客户端、工程、图形驱动、资源使用和官方客户端更新记录；不能仅凭白屏断定是 MCP 导致。
- **打开了很多 EDA 会话**：先用 `session list` 检查；不要用 `open` 来探测连接。关闭用户手动打开的窗口前，先确认工程保存且确实不再需要该窗口。

更详细的检查方式和官方链接见 [`references/setup-and-troubleshooting.md`](references/setup-and-troubleshooting.md)。

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
