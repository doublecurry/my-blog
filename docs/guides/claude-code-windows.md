---
title: 'A Complete Setup and Guide for Claude Code on Windows'
description: 'A practical Windows setup guide for Claude Code, terminal tooling, shell integration, and workflow polish.'
created: 2026-02-10
updated: 2026-05-13
tags:
  - Claude Code
  - Windows
  - Developer Workflow
---

# A Complete Setup and Guide for Claude Code on Windows

Due to the rapid update cycles of Claude Code, official documentation can sometimes lag behind in covering all edge cases and platform differences. This article will guide you from scratch to build a smooth, efficient, and visually appealing Claude Code environment on Windows, providing precise solutions for some obscure, high-frequency issues.

---

## I - Preparation and Basic Environment Setup

Before officially installing the various tools, ensuring a clear network environment is the top priority. It is recommended to configure a proxy in the terminal first to ensure the smooth pulling of subsequent dependencies.

Execute the following commands in PowerShell (please replace `Port` with your actual port):

```powershell
$env:HTTP_PROXY="http://127.0.0.1:Port"
$env:HTTPS_PROXY="http://127.0.0.1:Port"

```

Next, we will install a series of core components sequentially to enhance the development experience. If you are unsure about the specific role of a tool, simply execute the installation in order.

### 1. Terminal and Package Manager

* **Windows Terminal**: A powerful modern terminal. After installation, it is recommended to set the system's built-in PowerShell as the default profile and run it with administrator privileges by default. This greatly improves the experience of right-clicking "Open in Terminal" within folders later.
* **WinGet**: The official package manager for Windows. Use the following commands to repair or initialize it:
```powershell
$progressPreference = 'silentlyContinue'
Install-PackageProvider -Name NuGet -Force | Out-Null
Install-Module -Name Microsoft.WinGet.Client -Force -Repository PSGallery | Out-Null
Write-Host "Using Repair-WinGetPackageManager cmdlet to bootstrap WinGet..."
Repair-WinGetPackageManager -AllUsers

```



### 2. Core Shell and Editor

* **PowerShell 7**:
```powershell
winget install Microsoft.PowerShell

```


After installation, change the default shell of Windows Terminal to PowerShell 7 and check "Run as administrator." You must **close all windows and open a new terminal** at this point.
* **Notepad 4** (Lightweight Editor):
```powershell
winget install zufuliu.notepad4

```


After installation, you can replace the default Notepad in the system integration settings.

### 3. Version Control and Node.js Environment

* **Git for Windows**:
```powershell
winget install --id Git.Git -e --source winget

```


* **fnm (Fast Node Manager)**: An efficient Node version management tool.
```powershell
winget install Schniz.fnm

```


After installation, restart the terminal and execute the following commands to install and use an LTS version (e.g., Krypton):
```powershell
fnm install lts/krypton
fnm use lts/krypton

```


**Configure Environment Variables**: Enter `notepad $profile`. If it prompts that the file does not exist, first run `New-Item -Path $Profile -Type File -Force` to create it. Append the following content to the opened file and save:
```powershell
fnm env --use-on-cd --shell powershell | Out-String | Invoke-Expression

```



### 4. Install Claude Code

Restart the terminal, apply the newly set Node environment, switch the npm mirror registry, and install globally:

```powershell
fnm use lts/krypton
npm config set registry https://registry.npmmirror.com
npm install -g @anthropic-ai/claude-code@2.1.112

```

---

## II - Terminal Beautification (Optional Advanced Setup)

A visually pleasing terminal can significantly boost coding happiness.

* **Change to a Monospaced Font**: It is recommended to use the Maple Mono font, which supports ligatures and Chinese characters. In the PowerShell profile -> Appearance settings, change the font to `Maple Mono Normal NF CN`.
* **Configure Oh My Posh**:
```powershell
winget install JanDeDobbeleer.OhMyPosh --source winget --scope user --force

```


Open the profile again with `notepad $profile` and append the following command to activate the Oh My Posh theme:
```powershell
oh-my-posh init pwsh --eval | Invoke-Expression

```


You can pick your preferred style from the Oh My Posh theme library.

---

## III - High-Frequency Troubleshooting (FAQ)

### Q1: How do I switch to Plan Mode on Windows?

* On Node v24.2.0 or v22.17.0 and above, use **Shift + Tab** normally to switch.
* On older, unsupported Node versions, the shortcut will automatically fall back and bind to **Alt + M**.
* *Note: Version 2.0.31 and newer built with bun do not have this issue; it is unified as `Shift + Tab` across all platforms.* ### Q2: How do I paste images in the terminal?
* **Windows Environment**: Starting from version 1.0.93, support for the **Alt + V** shortcut was added. It can now read image data directly from the clipboard, not just image files.
* **macOS Environment**: Starting from version 1.0.61, the paste shortcut has been optimized to the native **⌘ + V**.

### Q3: Why does pressing enter in the VSCode integrated terminal add an extra backslash `\`?

This is a historical compatibility issue. Open `keybindings.json` and change the binding content from `"\\\r\n"` to `"\u001B\u000A"`. This will make the behavior consistent with the standard `⌥Option + Enter`.

### Q4: Getting an `Error: cannot open _claude_fs_right` exception popup?

This issue usually occurs during the path conversion process in the VSCode collaborative editing phase.
**Solution**: Temporarily disable the IDE Auto Connect feature, or temporarily uninstall the corresponding VSIX extension in VSCode. According to the latest tests, running it independently through the PowerShell terminal process mentioned above bypasses this issue, and the official team has significantly fixed IDE connection stability since version 1.0.65.

### Q5: MCP Server (Stdio) completely unusable on Windows?

You need to adjust the structure parameters of the calling command. Change the `command` field of the execution command to `"cmd"`, and pass `["/c", "npx"]` in the `arg` array so it parses correctly.

### Q6: Does the red "Offline" text at the bottom relate to account risk control/bans?

Completely unrelated. `Offline` only serves as a real-time network connection status prompt. The telemetry information collected by the system only contains basic metadata for troubleshooting and never involves ban detection.

### Q7: What are the configuration differences between official direct connection / third-party relays / custom proxy endpoints?

All related configurations only need to be modified under the `env` node in `~/.claude/settings.json`, **completely without touching the operating system's global environment variables or messing with TUN virtual network card proxies**.
*Please note: The priority of `ANTHROPIC_API_KEY` is always higher than the official website's auto-login state. As long as this variable exists, the system will prioritize the custom endpoint.*

1. **Official Direct Connection**: After configuring `HTTP_PROXY` / `HTTPS_PROXY`, log in and authorize normally.
2. **Standard Relay** (e.g., Moonshot K2 endpoint):
* `"ANTHROPIC_BASE_URL"`: Fill in the standard Anthropic format endpoint address.
* `"ANTHROPIC_API_KEY"`: Fill in your exclusive Key.


3. **NewAPI Conversion Proxy** (e.g., Aηyrouter):
* `"ANTHROPIC_BASE_URL"`: Fill in the provided conversion endpoint address.
* `"ANTHROPIC_AUTH_TOKEN"`: Fill in the authentication Token (usually in `sk-xxxxxx` format). This item and API_KEY are most likely mutually exclusive and cannot coexist.


4. **Quick Multi-Configuration Management**: Version 1.0.61 and above supports starting with the `--settings` parameter. You can copy multiple `.json` files with different configurations to achieve a seamless, one-click API source switch.