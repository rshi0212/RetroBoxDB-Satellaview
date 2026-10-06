# RetroBoxDB Satellaview

English | [中文说明](README.zh-CN.md)

Single-file SQLite preservation database for Nintendo Satellaview (BS-X). The public Catalog holds metadata only (checksums, DAT and provenance records, header fields, archive recipes and the processing code); it contains no ROM data and cannot restore files. The populated database stays local.

| Item | Value |
| --- | --- |
| Original size | 739 source ZIPs, 324.1 MiB (No-Intro 699, RetroAchievements sets 40); 739 ROM files, 703.6 MiB uncompressed |
| Stored size | populated database 115.9 MiB; public Catalog 10.6 MiB (no ROM data) |
| Ratio | 35.8% of the source ZIPs, 16.5% of the uncompressed ROM files |
| Technology | storage v4: SHA256-deduplicated 32 KiB blocks packed in No-Intro family order into solid LZMA2 groups of up to 256 MiB (256 MiB dictionary); per-block SHA256 and per-object CRC32/MD5/SHA1/SHA256 verification; source ZIPs reproduced byte-for-byte from TorrentZip plans |
| Export performance | Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, Python 3.14.4, all checks included. whole newest-DAT set with `export_set.py` (561 files, each checked against the DAT hashes): 61.6 MiB/s, 15 ms per file on average; single file with a cold cache (the group is decoded up to the file): ROM 2.78 s, TorrentZip 2.613 s on average |

## Downloads and documents

| File / document | Content |
| --- | --- |
| [RetroBoxDB.Satellaview.Catalog.sqlite](https://github.com/rshi0212/RetroBoxDB-Satellaview/releases/latest/download/RetroBoxDB.Satellaview.Catalog.sqlite) | Public Catalog (Release asset with `SHA256SUMS`) |
| [Storage v4 guide](RetroBoxDB.Storage-v4.en.md) / [中文](RetroBoxDB.Storage-v4.zh-CN.md) | Storage evaluation, contents, RA, names and maintenance for every platform |
| [Technical design](RetroBoxDB.Storage-v4.Technical-Design.en.md) | Storage format, platform adapters, incremental updates, verification |
| [RA list](reports/ra-satellaview-games.csv) / [summary](reports/ra-satellaview.json), [build report](reports/satellaview-build-report.json), [audit resolution](reports/audit-resolution-20261004.md) | Detailed data |

## Storage choice and platform specifics

Change against 32 MiB groups on real data (whole collection with 16 KiB blocks, 256 MiB unique): 64 MiB −1.28%, 128 MiB −3.33%, 256 MiB −6.32%; 256 MiB chosen by the rule.

- BS-X memory-pack images broadcast for the Satellaview, plus the BS-X base cartridge. Most packs are 1 MiB and many broadcasts repeat content or padding, so block deduplication removes about 60% of the data before compression; 32 KiB blocks in 256 MiB groups measured smallest on the whole collection.
- The BS header at 0x7FB0 (LoROM) or 0xFFB0 (HiROM) is located by the fixed byte 0x33, the map mode and the checksum pair; maker code, program type, Shift-JIS title, block allocation, remaining starts, broadcast month/day, map mode, execution type and version are stored. Files without a BS header (data packs) stay `unclassified`; the base cartridge has a standard SNES header and is stored in `snes_hardware`.
- RetroAchievements has no Satellaview console: BS games are listed under SNES (console 3) and hashed with the SNES method. The `.bs` files of the RetroAchievements SNES folder are imported here, not into SNES. This database's RA report covers only RA games tied to its own ROMs, DAT entries or DB Export files; SNES games are reported by RetroBoxDB-SNES.
- No Chinese name source exists yet; names can be added later with `tools/update_db.py --names`.

## Contents

| Item | Value |
| --- | --- |
| ROM records / games / releases | 602 / 606 / 609 |
| DAT coverage per version | 20260619-093425: 569/603; 20260814-103513: 566/604; 20260919-025009: 562/610 |
| Local ROMs in no DAT | 34 |
| ROM files of the RetroAchievements set | in a No-Intro DAT 22, RA only 18, hash not in the latest RA snapshot 0 ([list](reports/ra-satellaview-collection-unknown.csv)); RA games still without a local ROM: [gap list](reports/ra-satellaview-missing.csv) |
| No-Intro DB Export + Dump Log 20260919-025009 | 615 archives, 766 file identities, 6 documented hardware assertions; Dump Log Verified 7 |
| RetroAchievements (console 3) | 13 games with achievements: 13 with a local ROM (44 ROMs), 0 with the ROM in a sibling database, 0 DAT only, 0 DB file only, 0 without a No-Intro counterpart |
| Chinese names | 0 of 0 rows translated (0 unique); 0 local ROMs have a Chinese name |
| Populated-database audit | 607 objects, 2 groups, 923 archive plans, all passed |

Every source ZIP is reproduced byte-for-byte from its TorrentZip plan (`v_file_checksums.exported_bytes_equal_source`).

## Usage

```bash
# Query-only audit with the Catalog's embedded engine (also: stats, checksums FILE_ID, help)
python3 -B -c 'import sqlite3,sys; c=sqlite3.connect(sys.argv[1]); s=c.execute("SELECT content FROM resources WHERE name=?",("engine.py",)).fetchone()[0]; c.close(); exec(compile(s,"RetroBoxDB:engine.py","exec"))' ./RetroBoxDB.Satellaview.Catalog.sqlite audit
# Populated database: export by DAT version, 1G1R, RA achievements, TorrentZip or plain ROMs
python3 -B tools/export_set.py RetroBoxDB.Satellaview.sqlite OUT --set 1g1r --ra achievements --container torrentzip --layout ra-category
# Add new DATs, DB Export / Dump Log snapshots, ROMs and an RA snapshot incrementally
python3 -B tools/update_db.py RetroBoxDB.Satellaview.sqlite --discover --ra --catalog RetroBoxDB.Satellaview.Catalog.sqlite
```

Python 3.10+ standard library only. `engine.py` and the other `resources` entries are executable code; run them only from a database you built or a Release asset whose SHA256 you verified. Releases are produced by `.github/workflows/publish-catalog.yml`: it starts from the base Catalog pinned in `release/catalog-release.json`, injects the engine and documents of this commit, checks every data-table digest, runs the tests and the audit, then publishes.
