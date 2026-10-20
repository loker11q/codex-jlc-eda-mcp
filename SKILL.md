---
name: codex-jlc-eda-mcp
description: Connect Codex to the JLCEDA Professional desktop client through its local CLI/MCP bridge, verify the connection, and safely inspect or automate schematic and PCB work. Use when configuring or troubleshooting this integration, or when the user asks Codex to operate JLCEDA Professional through MCP.
---

# Codex + JLCEDA Professional

Use the JLCEDA Professional desktop client's built-in command-line interface (CLI) and local bridge to connect Codex to an already-running editor. MCP means Model Context Protocol; it exposes the editor's supported operations as tools.

For exact setup fields and troubleshooting, read [references/setup-and-troubleshooting.md](references/setup-and-troubleshooting.md). Check the official JLCEDA CLI documentation when the installed version, command syntax, or API behavior differs from the reference.

## Connection workflow

1. Confirm the user is using the JLCEDA Professional **desktop client**, then locate its executable. Do not assume the install path. On Windows the command is usually `lceda-pro.exe`; other editions may use another executable name.
2. Check the installed client's `--help` output and current official CLI requirements. The built-in CLI/MCP feature is version-dependent. Do not download or upgrade software on the user's behalf.
3. Configure Codex with the executable as the command and `--mcp` and `stdio` as **two separate argument values**. Each item in the MCP `args` array is one argv token. Do not put `--mcp stdio` into one argument field. Set a valid working directory, preferably the executable's directory; leave environment variables empty unless the user needs them.
4. Ask the user to restart/reload the Codex MCP connection after changing settings. Do not launch extra EDA windows just to test the MCP server.
5. Verify in order: inspect Codex's MCP status (for example `/mcp` where supported), run the executable's read-only `doctor`, and use a read-only MCP operation such as listing sessions or reading the active project summary. Confirm the expected project/session before making changes.
6. If startup repeatedly opens GUI windows, disable the MCP entry first. Recheck that its command is the CLI executable and that its arguments are separate tokens before re-enabling it. Never keep retrying a malformed launcher.

## Design task workflow

- Before touching a design, identify the exact project and requested outputs (schematic, PCB, checks, exports). If the request could refer to multiple projects or windows, inspect and ask which one.
- Treat explicit requests to create or modify a named design as authorization for that scoped work. Do not overwrite another project, change unrelated documents, or generate manufacturing exports unless requested.
- For electrical designs, establish the requirements that affect correctness: input range, output voltage and current, topology, load/transient needs, thermal constraints, connector/polarity, and board constraints. If key requirements are missing, ask concise questions or state conservative assumptions and get agreement when those assumptions materially change the circuit.
- Use JLCEDA's supported API/MCP tools and component libraries to create or edit design objects. Do not invent API methods, pin coordinates, symbols, footprints, or component availability. Read API documentation/tool schemas when needed and retrieve pin coordinates from the editor API.
- Keep changes inside the selected project. Preserve an existing project unless the user explicitly requests editing it; for a new project, choose a distinct name and check for conflicts first.
- After each important edit, verify the response indicates success and inspect the resulting editor state. Run relevant schematic/PCB checks (such as ERC/DRC), inspect netlist and BOM consistency, and verify important geometry. An API response showing that code ran does not by itself prove the design operation succeeded.
- Do not claim a circuit or PCB is production-ready solely because it was drawn or passed automated checks. Report assumptions, checks performed, unresolved warnings, and exported file paths. Save/export only as requested.
- Avoid creating or closing editor sessions unless required. Never close a user-opened session or discard unsaved edits without first confirming the project is saved and the user wants that window closed.

## Minimal user-facing test

After setup, ask Codex to use the configured JLCEDA MCP server to list current sessions and report whether the intended project is open. Explicitly request a read-only test that does not modify or save the design. If the server is missing, inspect Codex's MCP configuration/status and restart the client before trying again.
