Satellaview Catalog, storage v4 (32 KiB blocks, 2 solid LZMA2 groups of up to 256 MiB). Metadata only: **no ROM payloads are published**; `compression_groups`, `chunks` and `object_chunks` are empty.

- One schema for all twenty-three platforms: the header tables of the eight new platforms (Game Gear, PC Engine, SuperGrafx, MSX, MSX2, Virtual Boy, Game & Watch, Super A'Can: `pce_hardware`, `msx_hardware`, `vb_hardware`) and `rom_annotations` exist in every Catalog; tables of other platforms and provider tables have no rows.
- RetroAchievements reports look up sibling databases (NES<->FDS, SNES<->Satellaview, WonderSwan<->WonderSwan Color, NeoGeo Pocket<->NeoGeo Pocket Color, PC Engine<->SuperGrafx, MSX<->MSX2): a game whose ROM is stored there is `local_other_platform`, not a gap.
- Source: 739 ZIPs (nointro 699, retroachievements 40), 324.1 MiB (739 ROM files, 703.6 MiB uncompressed). Populated database: 116.1 MiB (35.8% of the ZIPs). All source ZIPs are reproduced byte-for-byte.
- Contents: 602 ROM records, 606 games, 609 releases; DAT versions: 20260619-093425, 20260814-103513, 20260919-025009.
- RetroAchievements: 13 of 13 games with achievements have a local ROM.
- Export (Intel(R) Core(TM) i7-8650U CPU @ 1.90GHz, idle, all checks): whole newest-DAT set with export_set.py 61.6 MiB/s (561 files); single file with a cold cache 2.78 s (ROM) / 2.613 s (TorrentZip) on average.
- Full audit of the populated database: 607 objects, 2 groups, 923 archive plans, no errors.

The release workflow starts from the base Catalog pinned by SHA256 in `release/catalog-release.json`, injects the engine and documents of the tagged commit, checks every data-table digest, SQLite integrity and foreign keys, runs the Catalog audit and the repository tests. Verify the download with `SHA256SUMS`.

[中文说明](https://github.com/rshi0212/RetroBoxDB-Satellaview/blob/main/README.zh-CN.md)
