# FastCopyManager

English | [简体中文](#简体中文) | [日本語](#日本語) | [한국어](#한국어) | [Français](#français)

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

---

<a id="日本語"></a>
## 日本語

FastCopyManager は [FastCopy](https://fastcopy.jp/) を使いやすくするための補助ツールです。複数のバックアッププロファイルを管理するグラフィカルインターフェースを提供し、ローカルの FastCopy を呼び出してバックアップを実行します。

私と同じように、複数のフォルダ間で定期的なバックアップを行う必要があり、FastCopy の「毎回ソースと宛先を手動で選ぶ」作業が面倒だと感じているなら、このツールはあなたのためのものです。よく使うバックアップタスクをプロファイルとして保存し、ワンクリックまたはバッチキューで実行できます。

### 機能

- **プロファイル管理**: バックアッププロファイルの作成 / 編集 / 複製 / 名前変更 / 削除をカードリストで表示
- **複数のバックアップモード**: 差分、更新、強制（完全コピー）、新規のみ、同期、移動（6 モード、下表参照）
- **バッチ実行キュー**: 複数のプロファイルをキューに追加し、バッジ順に順次実行、いつでも停止可能
- **クイック実行**: 各カードに「今すぐ実行」ボタン — 詳細ページを開かずに実行
- **ドラッグ & ドロップ**: フォルダ / ファイルをソースリストにドラッグ；カードをドラッグして並べ替え
- **柔軟なフィルタリング**: 含める / 除外ルール（`*` `?` ワイルドカード）、ファイルサイズの上下限、速度制限、コピー後検証
- **日付範囲フィルター**: 指定した日付範囲内に変更されたファイルのみバックアップ（「昨年」「今年」の便利なショートカット付き）
- **スケジュールバックアップ**: プロファイルごとにスケジュール（毎日 / 毎週 / 毎月 / ログオン時、遅延設定可）を Windows タスクスケジューラで実行
- **スタンドアロンスクリプトエクスポート**: ワンクリックでプロファイルから自己完結型の PowerShell + BAT スクリプトを生成 — 本アプリ不要で単独実行可能、任意のスケジューラから起動可能
- **ターゲットオフライン時の自動スキップ**: 宛先ドライブが見つからない場合（USB ドライブが抜かれたなど）、誤った失敗を報告する代わりに通知付きで実行をスキップ
- **バックアップ履歴**: 実行ごとに時間 / 結果 / 所要時間を記録；エントリをダブルクリックしてログを開く
- **ユーザー向けログ**: 各バックアップは独立した `.log` ファイルを宛先フォルダに書き込み — ログはバックアップと共に移動
- **ポータブル設計**: 単一のグリーン exe；すべての設定とデータは exe の隣に保存 — コピーするだけで移行完了
- **多言語 UI**: 中文 / English / 日本語 / 한국어 / Français；デフォルトでシステム言語に従い、アプリ内で切り替え可能

### 動作環境

| 項目 | 要件 |
|---|---|
| OS | Windows 10 / 11（x64） |
| ランタイム | [.NET 8 Desktop Runtime](https://dotnet.microsoft.com/download/dotnet/8.0)（exe に同梱されていません） |
| バックアップエンジン | [FastCopy](https://fastcopy.jp/) がローカルにインストール済み（デフォルトパス `C:\Programs\FastCopy\FastCopy.exe`、設定で変更可） |

### クイックスタート

1. `FastCopyManager.exe` をダウンロードして任意のフォルダに配置（設定ファイルが exe の隣に作成されるため、専用フォルダを推奨）
2. .NET 8 Desktop Runtime と FastCopy がインストールされていることを確認
3. 起動して「＋ 新規プロファイル」をクリックし、最初のバックアッププロファイルを作成
4. 名前を入力し、ソースを追加（ドラッグ & ドロップ対応）、宛先を選択して「▶ バックアップ実行」をクリック

### 使い方

#### メインウィンドウ

- **カードをダブルクリック**するか、右側の「編集」をクリックしてプロファイル詳細を開く
- **カードをドラッグ**してプロファイルを並べ替え
- 左下: FastCopy.exe のパス設定、UI 言語の切り替え

#### バックアップモード

| モード | 動作 |
|---|---|
| 差分 | 宛先にないファイル、またはサイズ / タイムスタンプが異なるファイルをコピー；ソースを優先 |
| 更新 | ソースが新しいか存在しないファイルのみコピー；古いソースで上書きしない |
| 強制 | 宛先の状態に関わらずすべてのファイルを再コピー |
| 新規のみ | 宛先にないファイルのみコピー；既存ファイルは触らない |
| 同期 | 宛先をソースと完全に同一に: 差分をコピーし、**ソースにない宛先ファイルを削除**（危険） |
| 移動 | コピー後にソースファイルを削除、切り取り＆貼り付けのような動作（危険） |

> ⚠️ 「同期」と「移動」は破壊的操作です。実行前に確認を求められます。

#### 実行キュー

複数のバックアップを連続して実行するには:

1. プロファイルを選択して「キューに追加」をクリック — 青いバッジに実行順が表示
2. 繰り返してさらにプロファイルを追加
3. 「キュー実行 (N)」をクリック；バッジ順に実行され、「停止」ボタンでいつでも中断可能

#### スケジュールバックアップ

各プロファイルに独自のスケジュールを設定できます:

1. プロファイルを開き「⏰ スケジュール」をクリック
2. 「スケジュールバックアップを有効にする」にチェックし、頻度を選択: **毎日 / 毎週 / 毎月 / ログオン時**（「ログオン時」は遅延を設定可、デフォルト 30 分で起動を妨げない）
3. 「保存」をクリック — すぐに有効化

スケジュールは **Windows タスクスケジューラ**に登録され（タスク名 `\FastCopyManager\<profile-id>`）、アプリが開いていなくても実行されます。管理者権限は不要です。その時点で宛先ドライブが利用不可の場合（USB ドライブが抜かれているなど）、実行は**スキップ**されトレイ通知が表示され、履歴には失敗ではなく「スキップ」と記録されます。

#### スタンドアロンスクリプトの生成

別のスケジューラからバックアップを実行したい、または正確なコマンドのコピーを残したい場合:

1. プロファイルを開き「📜 スクリプト生成」をクリック
2. 保存先を選択すると、自己完結型の `FastCopyバックアップ-<name>-<date>.ps1` と対応する `.bat` が作成されます
3. `.bat` をダブルクリック（または任意のツールから呼び出し） — **FastCopyManager 不要**、FastCopy 本体さえあれば実行可能

スクリプトはアプリの動作を模倣します: 最初に宛先ドライブをチェックし（オフラインならスキップ）、同じオプションで FastCopy を呼び出し、宛先フォルダに `BackupLog` を書き込みます。デフォルトでサイレント実行されます。FastCopy の進捗ウィンドウを表示したい場合は、スクリプト内の `/no_ui` を削除してください。

#### 宛先ルール

ソースフォルダは宛先ディレクトリに**まるごとコピー**されます。例えばソース `D:\photos`、宛先 `E:\backup` の場合、`E:\backup\photos\…` が生成されます。

#### バックアップログ

- バックアップ後、独立したログが**宛先フォルダ**に書き込まれます。ファイル名は `BackupLog-ProfileName-YYYYMMDDHHmmss.log`
- ログは人間が読むためのもので、アプリが読み戻すことはありません。「履歴」のエントリをダブルクリックして対応するログを開けます
- アプリ自身のプロファイルデータと履歴は exe の隣の `profiles.json` と `history.json` に保存されます

#### 除外ルール

複数のルールは**セミコロン**で区切ります。`*` と `?` のワイルドカードに対応:

```
*.tmp;*.bak;Thumbs.db
```

### データファイル（ポータブル移行）

| ファイル | 用途 |
|---|---|
| `profiles.json` | すべてのバックアッププロファイル、実行キュー、FastCopy パス、UI 言語 |
| `history.json` | バックアップ履歴レコード |
| `debug.log` | 診断ログ（問題報告時に添付してください） |

別のコンピュータに移す場合は、exe とこれらのファイルを一緒にコピーするだけです。再設定は不要です。

### よくある質問

**Q: 「FastCopy が見つかりません」と表示される？**
「⚙ 設定」をクリックして FastCopy.exe のフルパスを指定するか、「自動検出に戻す」をクリックしてデフォルトパスを使用してください。

**Q: 履歴エントリをダブルクリックしてもログが開けない？**
ログはバックアップの宛先フォルダにあります。そのフォルダが削除、移動、または名前変更されている場合、ログは見つかりません。履歴レコード自体はアプリに保持されます。

**Q: ソースを C: から D: に移動した — すべて再追加する必要がある？**
いいえ。詳細ページでソースパスを選択し、「✎ 編集」をクリックしてその場で変更できます。

### ビルド（開発者向け）

.NET 8 SDK が必要です:

```
dotnet build FastCopyManager.csproj -c Release
dotnet publish FastCopyManager.csproj -c Release -r win-x64 --self-contained false -p:PublishSingleFile=true -o publish
```

### 免責事項

本アプリは FastCopy を管理するためのグラフィカルフロントエンドに過ぎません。実際のバックアップは FastCopy によって実行されます。「同期」と「移動」は慎重に使用し、宛先フォルダをよく確認してください。本ソフトウェアの使用によって生じたデータ損失について、作者は責任を負いません。

### ライセンス

[MIT](LICENSE)

---

<a id="한국어"></a>
## 한국어

FastCopyManager는 [FastCopy](https://fastcopy.jp/)를 더 쉽게 사용할 수 있도록 설계된 보조 도구입니다. 여러 백업 프로필을 관리하는 그래픽 인터페이스를 제공하며, 로컬 FastCopy를 호출하여 백업 작업을 실행합니다.

저처럼 여러 폴더 간에 정기적인 백업을 해야 하고 FastCopy의 "매번 수동으로 원본과 대상을 선택하는" 작업이 지루하다면, 이 도구는 당신을 위한 것입니다. 자주 쓰는 백업 작업을 프로필로 저장한 뒤 한 번의 클릭이나 일괄 대기열로 실행하세요.

### 기능

- **프로필 관리**: 백업 프로필 생성 / 편집 / 복제 / 이름 변경 / 삭제, 카드 목록으로 표시
- **여러 백업 모드**: 차등, 갱신, 강제(전체 복사), 신규만, 동기화, 이동(여섯 가지 모드, 아래 표 참조)
- **일괄 실행 대기열**: 여러 프로필을 대기열에 추가하고 배지 순서대로 하나씩 실행, 언제든 중지 가능
- **빠른 실행**: 모든 카드에 "지금 실행" 버튼 — 상세 페이지를 열지 않고 바로 실행
- **끌어서 놓기**: 폴더 / 파일을 원본 목록으로 끌어다 놓기; 카드를 드래그하여 프로필 순서 변경
- **유연한 필터링**: 포함 / 제외 규칙(`*` `?` 와일드카드), 파일 크기 최소/최대, 속도 제한, 복사 후 검증
- **날짜 범위 필터**: 선택한 날짜 범위 내에서 수정된 파일만 백업 가능("작년" / "올해" 편리한 바로가기 포함)
- **예약 백업**: 프로필별 예약(매일 / 매주 / 매월 / 로그온 시, 지연 설정 가능)을 Windows 작업 스케줄러로 실행
- **독립 실행 스크립트 내보내기**: 한 번의 클릭으로 프로필에서 자체 포함된 PowerShell + BAT 스크립트 생성 — 이 앱 없이도 단독 실행 가능하며 어떤 스케줄러로도 트리거 가능
- **대상 오프라인 시 자동 건너뛰기**: 대상 드라이브가 없으면(예: USB 드라이브 분리) 잘못된 실패를 보고하는 대신 알림과 함께 실행을 건너뜀
- **백업 기록**: 모든 실행은 시간 / 결과 / 소요 시간과 함께 기록됨; 항목을 두 번 클릭하여 로그 열기
- **사용자 로그**: 각 백업은 대상 폴더에 독립적인 `.log` 파일을 기록 — 로그는 백업과 함께 이동
- **휴대용 설계**: 단일 그린 exe; 모든 설정과 데이터는 exe 옆에 저장 — 복사만 하면 이전 완료
- **다국어 UI**: 中文 / English / 日本語 / 한국어 / Français; 기본적으로 시스템 언어를 따르며 앱 내에서 전환 가능

### 요구 사항

| 항목 | 요건 |
|---|---|
| OS | Windows 10 / 11(x64) |
| 런타임 | [.NET 8 Desktop Runtime](https://dotnet.microsoft.com/download/dotnet/8.0)(exe에 포함되지 않음) |
| 백업 엔진 | 로컬에 설치된 [FastCopy](https://fastcopy.jp/)(기본 경로 `C:\Programs\FastCopy\FastCopy.exe`, 설정에서 변경 가능) |

### 빠른 시작

1. `FastCopyManager.exe`를 다운로드하여 아무 폴더에나 넣기(설정 파일이 exe 옆에 생성되므로 전용 폴더 권장)
2. .NET 8 Desktop Runtime과 FastCopy가 설치되어 있는지 확인
3. 실행 후 "＋ 새 프로필"을 클릭하여 첫 백업 프로필 만들기
4. 이름을 입력하고 원본 추가(끌어서 놓기 지원), 대상 선택 후 "▶ 백업 실행" 클릭

### 사용 방법

#### 메인 창

- **카드를 두 번 클릭**하거나 오른쪽의 "편집"을 클릭하여 프로필 상세 열기
- **카드를 드래그**하여 프로필 순서 변경
- 왼쪽 하단: FastCopy.exe 경로 설정, UI 언어 전환

#### 백업 모드

| 모드 | 동작 |
|---|---|
| 차등 | 대상에 없거나 크기 / 타임스탬프가 다른 파일 복사; 원본 우선 |
| 갱신 | 원본이 더 새롭거나 없는 파일만 복사; 오래된 원본은 덮어쓰지 않음 |
| 강제 | 대상 상태와 관계없이 모든 파일을 다시 복사 |
| 신규만 | 대상에 없는 파일만 복사; 기존 파일은 건드리지 않음 |
| 동기화 | 대상을 원본과 동일하게 만듦: 차이점 복사 및 **원본에 없는 대상 파일 삭제**(위험) |
| 이동 | 복사 후 원본 파일 삭제, 잘라내기 & 붙여넣기와 유사(위험) |

> ⚠️ "동기화"와 "이동"은 파괴적 작업입니다. 실행 전 앱에서 확인을 요청합니다.

#### 실행 대기열

여러 백업을 연속으로 실행하려면:

1. 프로필을 선택하고 "대기열에 추가" 클릭 — 파란 배지에 실행 순서 표시
2. 반복하여 더 많은 프로필 추가
3. "대기열 실행(N)" 클릭; 배지 순서대로 실행되며 "중지" 버튼으로 언제든 중단 가능

#### 예약 백업

각 프로필에 고유한 일정을 설정할 수 있습니다:

1. 프로필을 열고 "⏰ 예약" 클릭
2. "예약 백업 활성화"를 체크하고 빈도 선택: **매일 / 매주 / 매월 / 로그온 시**("로그온 시"는 지연 설정 가능, 기본 30분으로 시작 속도 저하 방지)
3. "저장" 클릭 — 즉시 적용

일정은 **Windows 작업 스케줄러**에 등록되며(작업 이름 `\FastCopyManager\<profile-id>`), 이 앱이 열려 있지 않아도 실행됩니다. 관리자 권한은 필요 없습니다. 그 시점에 대상 드라이브를 사용할 수 없으면(예: USB 드라이브 분리) 실행이**건너뛰어**지고 트레이 알림이 표시되며, 기록에는 실패가 아닌 "건너뜀"으로 표시됩니다.

#### 독립 실행 스크립트 생성

다른 스케줄러에서 백업을 실행하거나, 정확한 명령의 사본을 보관하려면:

1. 프로필을 열고 "📜 스크립트 생성" 클릭
2. 저장 위치를 선택하면 앱이 자체 포함된 `FastCopy백업-<name>-<date>.ps1`과 해당 `.bat`를 작성
3. `.bat`를 두 번 클릭(또는 어떤 도구로든 호출) — **FastCopyManager 필요 없음**, FastCopy 자체만 있으면 됨

스크립트는 앱의 동작을 그대로 반영합니다: 먼저 대상 드라이브를 확인(오프라인이면 건너뜀)하고 동일한 옵션으로 FastCopy를 호출한 뒤 대상 폴더에 `BackupLog`를 작성합니다. 기본적으로 자동 실행됩니다. FastCopy 진행 창을 보고 싶다면 스크립트 내의 `/no_ui`를 삭제하세요.

#### 대상 규칙

원본 폴더는 대상 디렉터리에**통째로 복사**됩니다. 예를 들어 원본 `D:\photos`, 대상 `E:\backup`이면 `E:\backup\photos\…`가 생성됩니다.

#### 백업 로그

- 각 백업 후 독립적인 로그가**대상 폴더**에 기록되며, 파일명은 `BackupLog-ProfileName-YYYYMMDDHHmmss.log`
- 로그는 사람이 읽기만 하며, 앱이 다시 읽지 않습니다. "기록"에서 항목을 두 번 클릭하면 해당 로그가 열립니다
- 앱 자체의 프로필 데이터와 기록은 exe 옆의 `profiles.json`과 `history.json`에 저장됩니다

#### 제외 규칙

여러 규칙은**세미콜론**으로 구분하며, `*`와 `?` 와일드카드를 지원합니다:

```
*.tmp;*.bak;Thumbs.db
```

### 데이터 파일(휴대용 이전)

| 파일 | 용도 |
|---|---|
| `profiles.json` | 모든 백업 프로필, 실행 대기열, FastCopy 경로, UI 언어 |
| `history.json` | 백업 기록 레코드 |
| `debug.log` | 진단 로그(문제 보고 시 첨부) |

다른 컴퓨터로 옮길 때는 exe와 이 파일들을 함께 복사하면 됩니다. 재설정이 필요 없습니다.

### 자주 묻는 질문

**Q: "FastCopy를 찾을 수 없습니다" 메시지?**
"⚙ 설정"을 클릭하고 FastCopy.exe의 전체 경로를 지정하거나, "자동 감지로 복원"을 클릭하여 기본 경로를 사용하세요.

**Q: 기록 항목을 두 번 클릭해도 로그가 열리지 않나요?**
로그는 백업 대상 폴더에 있습니다. 해당 폴더가 삭제, 이동 또는 이름 변경되었다면 로그를 찾을 수 없습니다 — 기록 레코드 자체는 앱에 보관됩니다.

**Q: 원본을 C:에서 D:로 옮겼습니다 — 모든 것을 다시 추가해야 하나요?**
아니요. 상세 페이지에서 원본 경로를 선택하고 "✎ 편집"을 클릭하여 제자리에서 수정하세요.

### 빌드(개발자용)

.NET 8 SDK가 필요합니다:

```
dotnet build FastCopyManager.csproj -c Release
dotnet publish FastCopyManager.csproj -c Release -r win-x64 --self-contained false -p:PublishSingleFile=true -o publish
```

### 면책 조항

이 앱은 FastCopy를 관리하기 위한 그래픽 프론트엔드일 뿐입니다. 실제 백업은 FastCopy가 수행합니다. "동기화"와 "이동"은 주의해서 사용하고 대상 폴더를 다시 확인하세요. 이 소프트웨어 사용으로 인한 데이터 손실에 대해 작성자는 책임을 지지 않습니다.

### 라이선스

[MIT](LICENSE)

---

<a id="français"></a>
## Français

FastCopyManager est un outil d'aide conçu pour rendre [FastCopy](https://fastcopy.jp/) plus facile à utiliser. Il fournit une interface graphique pour gérer plusieurs profils de sauvegarde et exécute les tâches de sauvegarde en appelant le FastCopy local.

Si, comme moi, vous devez effectuer des sauvegardes régulières entre plusieurs dossiers et que vous trouvez fastidieux le processus FastCopy qui consiste à « choisir à chaque fois la source et la destination », cet outil est fait pour vous : enregistrez vos tâches de sauvegarde fréquentes en tant que profils, puis exécutez-les en un clic ou sous forme de file d'attente par lots.

### Fonctionnalités

- **Gestion des profils** : créer / modifier / dupliquer / renommer / supprimer des profils de sauvegarde, présentés sous forme de liste de cartes
- **Plusieurs modes de sauvegarde** : Diff, Mise à jour, Forcé (copie complète), Nouveaux uniquement, Synchronisation, Déplacement (six modes, voir le tableau ci-dessous)
- **File d'exécution par lots** : ajouter plusieurs profils à une file d'attente, les exécuter un par un dans l'ordre des badges, s'arrêter à tout moment
- **Exécution rapide** : un bouton « Exécuter maintenant » sur chaque carte — pas besoin d'ouvrir la page de détails
- **Glisser-déposer** : glisser des dossiers / fichiers dans la liste des sources ; glisser des cartes pour réorganiser les profils
- **Filtrage flexible** : règles d'inclusion / exclusion (caractères génériques `*` `?`), taille de fichier min / max, limitation de vitesse, vérification après copie
- **Filtre par plage de dates** : sauvegarder éventuellement uniquement les fichiers modifiés dans une plage de dates choisie (avec des raccourcis pratiques « Année dernière » / « Cette année »)
- **Sauvegarde planifiée** : des plannings par profil (quotidien / hebdomadaire / mensuel / à l'ouverture de session, avec un délai) exécutés via le Planificateur de tâches Windows
- **Export de script autonome** : un clic génère un script PowerShell + BAT autonome à partir d'un profil — il s'exécute seul (sans cette application) et peut être déclenché par n'importe quel planificateur
- **Saut automatique lorsque la cible est hors ligne** : si le lecteur de destination est absent (par ex. une clé USB débranchée), l'exécution est ignorée avec une notification au lieu de signaler un échec trompeur
- **Historique des sauvegardes** : chaque exécution est journalisée avec heure / résultat / durée ; double-cliquez sur une entrée pour ouvrir son journal
- **Journaux utilisateur** : chaque sauvegarde écrit un fichier `.log` indépendant dans le dossier de destination — les journaux suivent vos sauvegardes
- **Conception portable** : un seul exe vert ; tous les paramètres et données vivent à côté de l'exe — copiez-les et vous êtes migré
- **Interface multilingue** : 中文 / English / 日本語 / 한국어 / Français ; suit par défaut la langue du système, modifiable dans l'application

### Configuration requise

| Élément | Exigence |
|---|---|
| Système d'exploitation | Windows 10 / 11 (x64) |
| Runtime | [.NET 8 Desktop Runtime](https://dotnet.microsoft.com/download/dotnet/8.0) (non fourni avec l'exe) |
| Moteur de sauvegarde | [FastCopy](https://fastcopy.jp/) installé localement (chemin par défaut `C:\Programs\FastCopy\FastCopy.exe`, configurable dans les Paramètres) |

### Démarrage rapide

1. Téléchargez `FastCopyManager.exe` et placez-le dans n'importe quel dossier (un dossier dédié est recommandé — les fichiers de config sont créés à côté de l'exe)
2. Assurez-vous que .NET 8 Desktop Runtime et FastCopy sont installés
3. Exécutez-le et cliquez sur « ＋ Nouveau profil » pour créer votre premier profil de sauvegarde
4. Saisissez un nom, ajoutez des sources (glisser-déposer pris en charge), choisissez une destination, puis cliquez sur « ▶ Exécuter la sauvegarde »

### Utilisation

#### Fenêtre principale

- **Double-cliquez sur une carte** ou cliquez sur « Modifier » à sa droite pour ouvrir les détails du profil
- **Faites glisser les cartes** pour réorganiser les profils
- En bas à gauche : définir le chemin de FastCopy.exe, changer la langue de l'interface

#### Modes de sauvegarde

| Mode | Comportement |
|---|---|
| Diff | Copie les fichiers absents de la destination, ou différents en taille / horodatage ; la source l'emporte |
| Mise à jour | Copie uniquement les fichiers dont la source est plus récente ou absente ; les sources plus anciennes ne écrasent jamais |
| Forcé | Recopie tous les fichiers quel que soit l'état de la destination |
| Nouveaux uniquement | Copie uniquement les fichiers absents de la destination ; les fichiers existants ne sont pas touchés |
| Synchronisation | Rend la destination identique à la source : copie les différences et **supprime les fichiers de destination absents de la source** (dangereux) |
| Déplacement | Supprime les fichiers source après la copie, comme un couper-coller (dangereux) |

> ⚠️ « Synchronisation » et « Déplacement » sont destructifs. L'application demande confirmation avant de les exécuter.

#### File d'exécution

Pour exécuter plusieurs sauvegardes à la suite :

1. Sélectionnez un profil et cliquez sur « Ajouter à la file » — un badge bleu indique son ordre d'exécution
2. Répétez pour ajouter d'autres profils
3. Cliquez sur « Exécuter la file (N) » ; les profils s'exécutent dans l'ordre des badges, avec un bouton « Arrêter » pour interrompre à tout moment

#### Sauvegarde planifiée

Chaque profil peut avoir son propre planning :

1. Ouvrez un profil et cliquez sur « ⏰ Planning »
2. Cochez « Activer la sauvegarde planifiée » et choisissez une fréquence : **Quotidien / Hebdomadaire / Mensuel / À l'ouverture de session** (pour « à l'ouverture » vous pouvez définir un délai, par défaut 30 minutes, pour ne pas ralentir le démarrage)
3. Cliquez sur Enregistrer — cela prend effet immédiatement

Le planning est enregistré dans le **Planificateur de tâches Windows** (nom de la tâche `\FastCopyManager\<id-du-profil>`), il s'exécute même lorsque cette application n'est pas ouverte — et il ne nécessite pas de droits d'administrateur. Si le lecteur de destination est indisponible à ce moment (par ex. une clé USB débranchée), l'exécution est **ignorée** avec une notification de barre d'état, et l'historique affiche « Ignoré » plutôt qu'un échec.

#### Générer un script autonome

Pour piloter des sauvegardes depuis un autre planificateur, ou simplement garder une copie de la commande exacte :

1. Ouvrez un profil et cliquez sur « 📜 Générer un script »
2. Choisissez où l'enregistrer ; l'application écrit un `FastCopy-sauvegarde-<nom>-<date>.ps1` autonome ainsi qu'un `.bat` correspondant
3. Double-cliquez sur le `.bat` (ou laissez n'importe quel outil l'appeler) — **aucun FastCopyManager requis**, seulement FastCopy lui-même

Le script reflète le comportement de l'application : il vérifie d'abord le lecteur de destination (saut s'il est hors ligne), appelle FastCopy avec les mêmes options et écrit un `BackupLog` dans le dossier de destination. Il s'exécute silencieusement par défaut ; supprimez `/no_ui` à l'intérieur du script si vous préférez voir la fenêtre de progression de FastCopy.

#### Règle de destination

Les dossiers source sont copiés **entièrement dans** le répertoire de destination. Par exemple, une source `D:\photos` avec une destination `E:\backup` produit `E:\backup\photos\…`.

#### Journaux de sauvegarde

- Après chaque sauvegarde, un journal indépendant est écrit dans le **dossier de destination**, nommé `BackupLog-NomDuProfil-AAAAMMJJHHmmss.log`
- Les journaux sont destinés à la lecture humaine uniquement ; l'application ne les relit jamais. Double-cliquez sur une entrée de « Historique » pour ouvrir le journal correspondant
- Les propres données de profil et l'historique de l'application sont stockés dans `profiles.json` et `history.json` à côté de l'exe

#### Règles d'exclusion

Séparez plusieurs règles par des **points-virgules** ; les caractères génériques `*` et `?` sont pris en charge :

```
*.tmp;*.bak;Thumbs.db
```

### Fichiers de données (migration portable)

| Fichier | Utilité |
|---|---|
| `profiles.json` | Tous les profils de sauvegarde, file d'exécution, chemin FastCopy, langue de l'interface |
| `history.json` | Enregistrements de l'historique des sauvegardes |
| `debug.log` | Journal de diagnostic (joignez-le lors du signalement de problèmes) |

Pour passer à un autre ordinateur, copiez l'exe avec ces fichiers — aucune reconfiguration n'est nécessaire.

### FAQ

**Q : Message « FastCopy introuvable » ?**
Cliquez sur « ⚙ Paramètres » et indiquez le chemin complet de FastCopy.exe, ou cliquez sur « Restaurer la détection auto » pour utiliser le chemin par défaut.

**Q : Le double-clic sur une entrée de l'historique n'ouvre pas le journal ?**
Les journaux se trouvent dans le dossier de destination de la sauvegarde. Si ce dossier a été supprimé, déplacé ou renommé, le journal a disparu — l'enregistrement d'historique lui-même est conservé par l'application.

**Q : La source a été déplacée de C: vers D: — dois-je tout réajouter ?**
Non. Sélectionnez le chemin de la source dans la page de détails et cliquez sur « ✎ Modifier » pour le modifier sur place.

### Compilation (développeurs)

Nécessite le SDK .NET 8 :

```
dotnet build FastCopyManager.csproj -c Release
dotnet publish FastCopyManager.csproj -c Release -r win-x64 --self-contained false -p:PublishSingleFile=true -o publish
```

### Avertissement

Cette application n'est qu'une interface graphique pour gérer FastCopy ; la sauvegarde réelle est effectuée par FastCopy. Utilisez « Synchronisation » et « Déplacement » avec prudence et vérifiez bien votre dossier de destination. L'auteur n'est pas responsable de toute perte de données causée par l'utilisation de ce logiciel.

### Licence

[MIT](LICENSE)
