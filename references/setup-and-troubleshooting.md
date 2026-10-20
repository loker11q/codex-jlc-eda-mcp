# Setup and troubleshooting

This guide covers the local Codex desktop app and JLCEDA Professional desktop client. UI labels can change; use the installed app's current settings and the official documentation linked below.

## Windows setup

1. Start JLCEDA Professional and open the project you intend to test. Use the desktop client; the browser edition does not expose the local CLI bridge.
2. Locate the executable, commonly named `lceda-pro.exe`. Example only:

   ```text
   D:\嘉立创\lceda-pro1\lceda-pro.exe
   ```

3. In ChatGPT/Codex desktop settings, open **MCP servers** and add a local **STDIO** server:

   ```text
   Name: jlc_eda (or another short name)
   Command: D:\嘉立创\lceda-pro1\lceda-pro.exe
   Argument 1: --mcp
   Argument 2: stdio
   Working directory: D:\嘉立创\lceda-pro1
   ```

   Replace the example paths with the actual installation directory. In a TOML configuration the equivalent shape is:

   ```toml
   [mcp_servers.jlc_eda]
   command = 'D:\嘉立创\lceda-pro1\lceda-pro.exe'
   args = ["--mcp", "stdio"]
   cwd = 'D:\嘉立创\lceda-pro1'
   ```

   The important detail is that `args` is an array of tokens: `"--mcp"` and `"stdio"` are separate values. Do not enter `--mcp stdio` as a single argument value if the UI creates one argv item per row.

4. Save the server and restart/reload the Codex MCP connection as the UI requests. Check `/mcp` in a new/reloaded Codex conversation if the current conversation does not show newly added tools.
5. Run a minimal read-only prompt:

   ```text
   Use jlc_eda to list the current JLCEDA sessions and identify whether the project named <project> is open. Read only; do not modify or save anything.
   ```

   Replace `jlc_eda` with the configured server name. A successful test should return a tool result containing the expected live session/project. A generic statement that the server is configured is not sufficient evidence of an MCP tool call.

## CLI checks

Use the installed executable's full path if it is not on `PATH`:

```powershell
$app = 'D:\嘉立创\lceda-pro1\lceda-pro.exe'
& $app --help | Out-String
& $app doctor | Out-String
& $app session list | Out-String
```

- `doctor` should report `ok: true`, `connected: true` when the editor is running, and a compatible bridge/client version.
- `session list` is a read-only inventory. It does not needlessly open a new window. Review the list before operating when multiple projects or sessions are open.
- Some JLCEDA Windows CLI versions note that PowerShell does not capture their stdout normally. If the output appears blank, try piping to `Out-String` or inspect the terminal behavior described by that version's `--help`; do not assume blank output means the bridge is connected.
- `open` creates an editor session/window. Do not use it for a simple connection test when the target project is already open.

## Troubleshooting

### JLCEDA opens repeatedly or flashes new windows

1. Disable the `jlc_eda` MCP server in Codex settings and restart Codex to stop repeated launches.
2. Confirm the command is the actual JLCEDA desktop executable.
3. Confirm the two arguments are separate values (`--mcp`, then `stdio`). Remove accidental combined arguments, empty argument rows, or an incorrect executable path.
4. Set the working directory to an existing directory, such as the executable's folder.
5. Re-enable once, then verify the MCP status and make one read-only tool call. Do not repeatedly retry a launcher that opens the GUI.
6. If EDA still flashes while the MCP server is disabled and Codex is closed, investigate JLCEDA itself, the project, system compatibility, GPU driver/rendering, memory pressure, and the current client release notes.

The built-in MCP server is a local stdio process that communicates with the running editor through the local bridge. It is designed to expose editor operations as tools; a GUI window repeatedly appearing is a reason to inspect the launch command and argv first, but is not by itself proof of a particular root cause.

### MCP server is missing in Codex

- Ensure the MCP server is enabled and the client was restarted/reloaded after configuration changes.
- Verify command, arguments, and working directory. Use the full executable path if `lceda-pro` is not on `PATH`.
- Confirm JLCEDA Professional desktop client is installed and running. The online/browser edition does not provide this local session bridge.
- Run `doctor` and inspect its JSON status. Use `session list` to confirm the editor has a live session.
- If the Codex conversation predates the MCP change, start a new chat after the server is enabled.

### White screen or flicker continues

Separate EDA rendering symptoms from MCP startup: disable the server, restart Codex, and observe JLCEDA while no MCP server is running. If the symptom remains, review JLCEDA's client FAQ and release notes, check whether it occurs only in one project, and check GPU drivers/rendering and resource usage. Do not change project files as a first troubleshooting step.

## Official documentation

- JLCEDA Professional CLI: <https://prodocs.lceda.cn/cn/api/guide/cli.html>
- JLCEDA Professional client FAQ: <https://prodocs.lceda.cn/cn/faq/client/>
- JLCEDA Professional release notes: <https://prodocs.lceda.cn/cn/faq/update-record/>
- Codex MCP setup: <https://learn.chatgpt.com/docs/extend/mcp?site_variant=chatgpt&translationFallback=zh-Hans>
- Codex MCP configuration reference: <https://learn.chatgpt.com/docs/config-file/config-reference>
