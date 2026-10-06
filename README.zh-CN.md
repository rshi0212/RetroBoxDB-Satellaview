# RetroBoxDB Satellaview

[English](README.md) | 中文

任天堂 Satellaview（BS-X 卫星广播）的单文件 SQLite 保存库。公开的 Catalog 只含元数据（校验值、DAT 与来源记录、头部字段、打包配方和程序），不含 ROM 数据，不能独立恢复文件；完整库保留在本地。

| 项目 | 数值 |
| --- | --- |
| 原始大小 | 源 ZIP 739 个，324.1 MiB（No-Intro 699 个，RetroAchievements 集合 40 个）；解压后 ROM 739 个，703.6 MiB |
| 入库后大小 | 完整库 116.1 MiB；公开 Catalog 10.7 MiB（不含 ROM 数据） |
| 比例 | 完整库为原 ZIP 的 35.8%，为解压后 ROM 总量的 16.5% |
| 使用的技术 | 存储 v4：32 KiB 块按 SHA256 去重，按 No-Intro 游戏族顺序装入最大 256 MiB 的 LZMA2 实体组（字典 256 MiB）；逐块 SHA256、逐对象 CRC32／MD5／SHA1／SHA256 校验；源 ZIP 由 TorrentZip 配方逐字节重建 |
| 导出性能 | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz，空闲负载，Python 3.14.4，含全部校验。按最新 DAT 整套导出（`export_set.py`，561 个文件，逐个按 DAT 哈希校验）：61.6 MiB/s，平均 15 毫秒／个；单个文件冷缓存（每次清空缓存，需解压所在组的前段）：ROM 平均 2.78 秒，TorrentZip 平均 2.613 秒 |

## 下载与说明

| 文件／文档 | 内容 |
| --- | --- |
| [RetroBoxDB.Satellaview.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-Satellaview/releases/latest/download/RetroBoxDB.Satellaview.Catalog.sqlite) | 公开 Catalog（Release 附件，附 `SHA256SUMS`） |
| [存储 v4 说明](RetroBoxDB.Storage-v4.zh-CN.md)／[English](RetroBoxDB.Storage-v4.en.md) | 各平台的存储评估、内容、RA、中文名与维护 |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | 存储格式、平台适配、增量更新、校验 |
| [RA 清单](reports/ra-satellaview-games.csv)／[汇总](reports/ra-satellaview.json)、[构建报告](reports/satellaview-build-report.json)、[审计处理](reports/audit-resolution-20261004.md) | 逐项数据 |

## 本平台的存储选择与特殊情况

真实全量数据（全集，16 KiB 块，去重后 256 MiB）上相对 32 MiB 组的变化：64 MiB −1.28%，128 MiB −3.33%，256 MiB −6.32%；按规则采用 256 MiB。

- Satellaview 卫星广播的 BS 记忆卡镜像，以及 BS-X 本体卡带。记忆卡多为 1 MiB，许多广播内容和填充重复，压缩前的块级去重即可去掉约 60% 的数据；全集实测 32 KiB 块、256 MiB 组最小。
- BS 头位于 0x7FB0（LoROM）或 0xFFB0（HiROM），按固定字节 0x33、映射模式和校验和补码定位；保存厂商代码、程序类型、Shift-JIS 标题、块分配位、剩余启动次数、广播月日、映射模式、执行类型和版本。没有 BS 头的文件（数据包）记为 `unclassified`；本体卡带是标准 SNES 头，存入 `snes_hardware`。
- RetroAchievements 没有 Satellaview 主机：BS 游戏列在 SNES 主机（3）下，哈希按 SNES 规则计算。RA SNES 目录中的 `.bs` 文件导入本库，不进入 SNES 库。本库的 RA 报告只统计与本库 ROM、DAT 条目或 DB Export 文件有关的 RA 游戏，SNES 游戏由 RetroBoxDB-SNES 统计。
- 目前没有中文名来源；以后可用 `tools/update_db.py --names` 增量导入。

## 内容

| 项目 | 数值 |
| --- | --- |
| ROM 记录／游戏组／发行版本 | 602／606／609 |
| 各版 DAT 覆盖 | 20260619-093425：569/603；20260814-103513：566/604；20260919-025009：562/610 |
| 不在任何 DAT 的本地 ROM | 34 |
| RetroAchievements 集合中的 ROM 文件 | DAT 中有 22，仅 RA 收录 18，哈希不在最新 RA 快照 0（[清单](reports/ra-satellaview-collection-unknown.csv)）；仍缺本地 ROM 的 RA 游戏见 [缺口清单](reports/ra-satellaview-missing.csv) |
| No-Intro DB Export＋Dump Log 20260919-025009 | 615 个档案、766 个文件身份、6 条有文档的硬件声明；Dump Log Verified 7 |
| RetroAchievements（console 3） | 有成就的游戏 13 个：本地有 ROM 13（44 个 ROM），ROM 在兄弟库中 0，仅 DAT 有 0，仅 DB 文件 0，无 No-Intro 对应 0 |
| 中文名 | 0 条记录中 0 条有中文（0 个唯一名）；本地 ROM 0 个有中文名 |
| 完整库审计 | 607 个对象、2 个组、923 个 ZIP 配方，全部通过 |

源 ZIP 均可由 TorrentZip 配方逐字节重建（`v_file_checksums.exported_bytes_equal_source`）。

## 使用

```bash
# 用 Catalog 内嵌引擎做只读审计（stats、checksums FILE_ID、help 同理）
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.Satellaview.Catalog.sqlite audit
# 完整库：按 DAT 版本、1G1R、RA 成就、TorrentZip／裸 ROM 组合导出
python3 -B tools/export_set.py RetroBoxDB.Satellaview.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# 增量加入新 DAT、DB Export／Dump Log、ROM 与 RA 快照
python3 -B tools/update_db.py RetroBoxDB.Satellaview.sqlite --discover --ra --catalog RetroBoxDB.Satellaview.Catalog.sqlite
```

只需 Python 3.10+ 标准库。`resources` 中的 `engine.py` 等是可执行代码，只应从自己构建或 SHA256 已核对的 Release 附件中执行。发布由 `.github/workflows/publish-catalog.yml` 完成：工作流从 `release/catalog-release.json` 固定的基础 Catalog 出发，注入本仓库提交中的引擎与文档，核对全部数据表摘要、运行测试与审计后发布。
