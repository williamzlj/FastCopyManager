# FastCopyManager

English | [简体中文](#简体中文)

FastCopyManager is a helper tool designed to make [FastCopy](https://fastcopy.jp/) easier to use. It provides a graphical interface for managing multiple backup profiles, and runs backup tasks by calling the local FastCopy.

If you, like me, need to run routine backups between multiple folders and find FastCopy's manual "pick source and destination every time" workflow tedious, this tool is for you: save your frequent backup tasks as profiles, then run them with one click or as a batch queue.

## Features

- **Profile management**: create / edit / duplicate / rename / delete backup profiles, presented as a card list
- **Multiple backup modes**: Diff, Update, Force (full copy), New-only, Sync, Move (six modes, see table below)
- **Batch execution queue**: add multiple profiles to a queue, run them one by one in badge order, stop anytime
- **Quick run**: a "Run Now" button on every card — no need to open the details page
- **Drag & drop**: drag folders / files into the source list; drag cards to reorder profiles
- **Flexible filtering**: include/exclude rules (`*` `?` wildcards), min/max file size, speed limit, verify after copy
- **Date range filter**: optionally back up only files modified within a chosen date range (with handy "Last year" / "This year" shortcuts)
- **Scheduled backup**: per-profile schedules (daily / weekly / monthly / at logon, with a delay) run via Windows Task Scheduler
- **Standalone script export**: one click generates a self-contained PowerShell + BAT script from a profile — it runs on its own (no need for this app) and can be triggered by any scheduler
- **Auto-skip when the target is offline**: if the destination drive is missing (e.g. a USB drive was unplugged), the run is skipped with a notification instead of reporting a misleading failure
- **Backup history**: every run is logged with time / result / duration; double-click an entry to open its log
- **User-facing logs**: each backup writes an independent `.log` file into the destination folder — logs follow your backups
- **Portable design**: single green exe; all settings and data live next to the exe — copy them and you're migrated
- **Multilingual UI**: 中文 / English / 日本語 / 한국어 / Français; follows system language by default, switchable in-app

## Requirements

| Item | Requirement |
|---|---|
| OS | Windows 10 / 11 (x64) |
| Runtime | [.NET 8 Desktop Runtime](https://dotnet.microsoft.com/download/dotnet/8.0) (not bundled with the exe) |
| Backup engine | [FastCopy](https://fastcopy.jp/) installed locally (default path `C:\Programs\FastCopy\FastCopy.exe`, configurable in Settings) |

## Quick Start

1. Download `FastCopyManager.exe` and put it in any folder (a dedicated folder is recommended — config files are created next to the exe)
2. Make sure .NET 8 Desktop Runtime and FastCopy are installed
3. Run it and click "＋ New Profile" to create your first backup profile
4. Enter a name, add sources (drag & drop supported), pick a destination, then click "▶ Run Backup"

## Usage

### Main window

- **Double-click a card** or click "Edit" on its right to open the profile details
- **Drag cards** to reorder profiles
- Bottom-left: set the FastCopy.exe path, switch UI language

### Backup modes

| Mode | Behavior |
|---|---|
| Diff | Copies files missing in the destination, or differing in size / timestamp; source wins |
| Update | Only copies files where the source is newer or missing; older sources never overwrite |
| Force | Re-copies all files regardless of destination state |
| New-only | Only copies files missing in the destination; existing files untouched |
| Sync | Makes the destination identical to the source: copies differences and **deletes destination files absent from the source** (dangerous) |
| Move | Deletes source files after copying, like cut & paste (dangerous) |

> ⚠️ "Sync" and "Move" are destructive. The app asks for confirmation before running them.

### Execution queue

To run several backups in a row:

1. Select a profile and click "Add to Queue" — a blue badge shows its run order
2. Repeat to add more profiles
3. Click "Run Queue (N)"; profiles run in badge order, with a "Stop" button to abort anytime

### Scheduled backup

Each profile can have its own schedule:

1. Open a profile and click "⏰ Schedule"
2. Tick "Enable scheduled backup" and choose a frequency: **Daily / Weekly / Monthly / At logon** (for "at logon" you can set a delay, default 30 minutes, so backups don't slow down startup)
3. Click Save — it takes effect immediately

The schedule is registered with **Windows Task Scheduler** (task name `\FastCopyManager\<profile-id>`), so it runs even when this app is not open — and it needs no administrator rights. If the destination drive is unavailable at that moment (e.g. a USB drive is unplugged), the run is **skipped** with a tray notification, and the history shows "Skipped" rather than a failure.

### Generate a standalone script

To drive backups from another scheduler, or simply keep a copy of the exact command:

1. Open a profile and click "📜 Generate Script"
2. Choose where to save it; the app writes a self-contained `FastCopy备份-<name>-<date>.ps1` plus a matching `.bat`
3. Double-click the `.bat` (or let any tool call it) — **no FastCopyManager required**, only FastCopy itself

The script mirrors the app's behaviour: it checks the destination drive first (skipping if offline), calls FastCopy with the same options, and writes a `BackupLog` into the destination folder. It runs silently by default; delete `/no_ui` inside the script if you'd rather see FastCopy's progress window.

### Destination rule

Source folders are copied **entirely into** the destination directory. For example, source `D:\photos` with destination `E:\backup` produces `E:\backup\photos\…`.

### Backup logs

- After each backup, an independent log is written into the **destination folder**, named `BackupLog-ProfileName-YYYYMMDDHHmmss.log`
- Logs are for human reading only; the app never reads them back. Double-click an entry in "History" to open the corresponding log
- The app's own profile data and history are stored in `profiles.json` and `history.json` next to the exe

### Exclude rules

Separate multiple rules with **semicolons**; `*` and `?` wildcards are supported:

```
*.tmp;*.bak;Thumbs.db
```

## Data files (portable migration)

| File | Purpose |
|---|---|
| `profiles.json` | All backup profiles, execution queue, FastCopy path, UI language |
| `history.json` | Backup history records |
| `debug.log` | Diagnostic log (attach it when reporting issues) |

To move to another computer, copy the exe together with these files — no reconfiguration needed.

## FAQ

**Q: "FastCopy not found" message?**
Click "⚙ Settings" and specify the full path to FastCopy.exe, or click "Restore Auto-detect" to use the default path.

**Q: Double-clicking a history entry fails to open the log?**
Logs live in the backup destination folder. If that folder was deleted, moved or renamed, the log is gone — the history record itself is kept by the app.

**Q: Source moved from C: to D: — do I have to re-add everything?**
No. Select the source path in the details page and click "✎ Edit" to modify it in place.

## Building (developers)

Requires the .NET 8 SDK:

```
dotnet build FastCopyManager.csproj -c Release
dotnet publish FastCopyManager.csproj -c Release -r win-x64 --self-contained false -p:PublishSingleFile=true -o publish
```

## Disclaimer

This app is only a graphical front-end for managing FastCopy; the actual backup is performed by FastCopy. Use "Sync" and "Move" with care and double-check your destination folder. The author is not liable for any data loss caused by using this software.

## License

[MIT](LICENSE)

---

<a id="简体中文"></a>
## 简体中文

FastCopyManager 是一个为方便用户使用 [FastCopy](https://fastcopy.jp/) 而设计的辅助工具，提供图形化界面，方便用户管理多个备份方案，软件调用本机 FastCopy 执行备份任务。

如果你和我一样，平时要在多个文件夹之间做例行备份，又不满足于 FastCopy 原版"每次手动选来源和目标"的操作方式，这个工具就是为你准备的：把常用的备份任务保存成方案，一键或按序列批量执行。

### 功能特性

- **方案管理**：新建 / 编辑 / 复制 / 重命名 / 删除备份方案，卡片式列表一目了然
- **多种备份模式**：差异复制、更新复制、完全复制、仅新文件、同步、移动（六种模式，详见下表）
- **批量执行序列**：把多个方案加入执行序列，按角标顺序依次自动执行，支持中途停止
- **快速执行**：每张卡片上的「立即备份」按钮，无需进详情页直接运行
- **拖拽操作**：把文件夹 / 文件直接拖进源列表；拖动卡片调整方案顺序
- **灵活的过滤**：包含 / 排除规则（支持 `*` `?` 通配符）、体积上下限、速度限制、复制后校验
- **日期范围过滤**：可只备份指定日期范围内修改过的文件（提供「最近一年」「今年」快捷按钮）
- **定时备份**：每个方案可单独设置每天 / 每周 / 每月 / 登录后（可设延迟）自动备份，由 Windows 任务计划程序执行
- **导出可执行脚本**：一键把方案生成自包含的 PowerShell + BAT 脚本，脱离本软件也能运行，可交给任何调度器触发
- **目标盘不在时自动跳过**：目标磁盘不可用（如 U 盘被拔掉）时自动跳过并弹提示，而不是误报失败
- **备份历史**：每次备份自动记录时间 / 结果 / 耗时，双击打开对应日志
- **用户日志**：每次备份在输出目录生成独立的 `.log` 文件，日志跟着备份走
- **便携设计**：绿色单文件，所有配置和数据都保存在 exe 同目录，拷走即迁移
- **多语言界面**：中文 / English / 日本語 / 한국어 / Français，默认跟随系统语言，可手动切换

### 环境要求

| 项目 | 要求 |
|---|---|
| 操作系统 | Windows 10 / 11（x64） |
| 运行时 | [.NET 8 Desktop Runtime](https://dotnet.microsoft.com/download/dotnet/8.0)（不随 exe 打包，需自行安装） |
| 备份引擎 | 本机安装 [FastCopy](https://fastcopy.jp/)（默认查找 `C:\Programs\FastCopy\FastCopy.exe`，可在设置中手动指定） |

### 快速开始

1. 下载 `FastCopyManager.exe`，放到任意目录（建议单独建一个文件夹，配置文件会生成在同目录）
2. 确保已安装 .NET 8 Desktop Runtime 和 FastCopy
3. 双击运行，点「＋ 新建方案」创建你的第一个备份方案
4. 填写方案名称、选择源目录 / 文件（可直接拖入）、选择目标目录，点「▶ 立即备份」

### 使用说明

#### 主界面

- **双击卡片**或点右侧「编辑」进入方案详情
- **拖动卡片**可以调整方案在列表中的顺序
- 底部左侧：设置 FastCopy.exe 路径、切换界面语言

#### 备份模式

| 模式 | 说明 |
|---|---|
| 差异复制 | 目标没有、或大小 / 修改时间和源不一致的都复制，以源为准覆盖 |
| 更新复制 | 只复制源比目标新、或目标不存在的文件；源更旧时不会拿旧文件倒回去 |
| 完全复制 | 无论目标如何，全部文件重新复制一遍 |
| 仅新文件 | 只复制目标不存在的文件，已存在的文件一律不动 |
| 同步复制 | 使目标与源完全一致：复制差异文件，并**删除目标中源没有的文件**（危险） |
| 移动 | 复制完成后删除源文件，相当于剪切粘贴（危险） |

> ⚠️ 「同步」和「移动」是破坏性操作，执行前软件会弹出确认框，请确认后再继续。

#### 执行序列

需要一次性跑多个备份时：

1. 选中方案，点「加入执行序列」，卡片左上角会出现蓝色角标显示执行顺序
2. 重复上述操作把更多方案加入序列
3. 点「执行备份序列（N）」，软件按角标顺序依次执行，底部出现「停止」按钮可随时中断

#### 定时备份

每个方案可以单独设置定时：

1. 打开方案，点「⏰ 定时备份」
2. 勾选「启用定时备份」，选择频率：**每天 / 每周 / 每月 / 登录后**（选「登录后」可设延迟，默认 30 分钟，避免拖慢开机）
3. 点「保存」即生效

定时任务注册在 **Windows 任务计划程序**中（任务名 `\FastCopyManager\<方案Id>`），软件不打开也会按时执行，且不需要管理员权限。若到点时目标磁盘不可用（例如 U 盘被拔掉），本次会**自动跳过**并弹托盘提示，历史记录显示「已跳过」而不是失败。

#### 导出可执行脚本

想把备份交给别的调度器，或只是想留一份准确的命令：

1. 打开方案，点「📜 生成可执行脚本」
2. 选择保存位置，软件会生成自包含的 `FastCopy备份-方案名-日期.ps1` 以及配套的 `.bat`
3. 双击 `.bat`（或让任何工具调用它）即可执行 —— **无需 FastCopyManager**，只要有 FastCopy 就行

脚本行为与软件一致：先检查目标盘（不可用则跳过）、用相同参数调用 FastCopy、在目标目录生成 `BackupLog`。脚本默认静默运行，想看到 FastCopy 进度窗口就把脚本里的 `/no_ui` 删掉。

#### 目标目录规则

源文件夹会**整个复制**到目标目录下，例如源为 `D:\photos`、目标为 `E:\backup`，备份后得到 `E:\backup\photos\…`。

#### 备份日志

- 每次备份完成后，在**目标目录**生成一份独立日志，文件名为 `BackupLog-方案名-年月日时分秒.log`
- 日志只供用户查看，软件不回读；方案详情页的「执行历史」双击即可打开对应日志
- 软件自身的方案配置和历史记录保存在 exe 同目录的 `profiles.json` 和 `history.json`

#### 排除规则

多条规则用**分号**分隔，支持 `*` 和 `?` 通配符，例如：

```
*.tmp;*.bak;Thumbs.db
```

### 数据文件（便携迁移）

| 文件 | 作用 |
|---|---|
| `profiles.json` | 全部备份方案、执行序列、FastCopy 路径、界面语言 |
| `history.json` | 备份历史记录 |
| `debug.log` | 程序诊断日志（排查问题时提供给开发者） |

换电脑时把 exe 连同这几个文件一起拷走即可，无需重新配置。

### 常见问题

**Q：提示找不到 FastCopy？**
点左下角「⚙ 设置」，手动指定 FastCopy.exe 的完整路径；或点「恢复自动寻找」使用默认路径。

**Q：双击历史记录打不开日志？**
日志文件存放在备份的输出目录里，如果输出目录被删除、移动或改名，日志就找不到了；备份记录本身仍保留在软件中。

**Q：想把源路径从 C 盘改成 D 盘，要重新添加吗？**
不用。在详情页选中该源路径，点「✎ 修改」直接改路径即可。

### 构建（开发者）

需要 .NET 8 SDK：

```
dotnet build FastCopyManager.csproj -c Release
dotnet publish FastCopyManager.csproj -c Release -r win-x64 --self-contained false -p:PublishSingleFile=true -o publish
```

### 免责声明

本软件仅是 FastCopy 的图形化辅助管理工具，备份行为由 FastCopy 完成。请谨慎使用「同步复制」和「移动」模式，操作前请确认目标目录内容，因使用本软件造成的任何数据丢失，作者不承担责任。

### 许可证

[MIT](LICENSE)
