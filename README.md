# ER605 v2.20 — artifacts for OpenWrt developers

Public repository with the dumps and diagnostic artifacts collected from a
**TP-Link Omada ER605 V2.20 (UN)** during a successful OpenWrt 23.05.0
installation and the subsequent recovery of a bricked unit.

## Device

| Field | Value |
|---|---|
| Model | TP-Link Omada ER605 V2.20 (UN) |
| SoC | MediaTek MT7621AT (880 MHz, 4 cores) |
| NAND | ESMT 128 MiB, 128 KiB erase blocks |
| RAM | 256 MiB DDR3 |
| Stock firmware | 2.2.5 Build 20240522 Rel.75860 |
| Current firmware | OpenWrt 23.05.0, r23497-6637af95aa |

## Releases / assets

- `er605-dev-package-full.tar.gz` — documentation, dumps (`mtd0`, `mtd1`,
  `mtd2`, `mtd4`, `mtd17`), configs, stock firmware 2.2.5, OpenWrt 23.05.0
  initramfs/sysupgrade images, boot logs, `sysupgrade successful` log,
  `bootcmd` persistence log, catalog of the 22 stock UBI volumes, board/UART
  photos, pentest reports and helper scripts. Includes `MANIFEST.sha256`.
- `er605-mtd3-stock-backup.tar.gz` — full 130,809,856-byte `mtd3` backup
  (the `firmware` partition), catalog and MD5 `f968d8cb84a0c4ab5b072b15f6ea57b1`.
- `er605-dev-package.tar.gz` — small summary package (dumps + report + catalog).

## Key findings

- Stock `firmware` UBI layout has **22 volumes** (`partition-table`,
  `support-list`, `device-info`, `extra-para`, `kernel`, `rootfs`, `database`,
  plus `.b` variants). `vid_hdr_offset` is **2048** on both stock and OpenWrt.
- `mtd2` (`factory`, 256 KiB) and `mtd4` (`panic-ops`, 2 MiB) are **100% 0xFF**
  on this unit — the undocumented `panic-ops` partition was empty.
- The stock recovery web UI only accepts `name="firmware"`; it ignores MIME
  types and extra fields, and always reports `Upgrade failed!` for a raw dump.
- The only reliable recovery path is the UART console: J1 has
  `3V3 / GND / RX / TX` (C3 on the left); R2 and R5 pads must be bridged
  individually; 115200 8N1, 3.3 V.
- After a successful `sysupgrade`, the stock U-Boot still looks for stock
  volumes; set:
  `setenv bootcmd 'ubi part firmware 2048; ubi read 0x80100000 kernel; bootm 0x80100000'`
  then `saveenv`.

## Scope

No credential is included. No stock firmware above 2.2.5 was used or shipped.
The `mtd3` backup is kept intact as the recovery path to stock.
