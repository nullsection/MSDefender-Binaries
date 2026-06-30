# msdefender

A static analysis workspace for Microsoft Defender binaries: the shipped
executables and DLLs plus official Microsoft symbols (PDBs) for the subset that
Microsoft publishes. Analysis only.

## Contents

```
msdefender/
  binaries/
    ProgramFiles_WindowsDefender/   inbox build   (C:\Program Files\Windows Defender)
    Platform_4.18.26050.15-0/       updated engine (ProgramData\...\Platform\<ver>)
    ProgramFiles_x86_WindowsDefender/  32-bit Defender set (8 files)
    Drivers/                        kernel drivers: WdFilter, WdBoot, WdDevFlt, WdNisDrv (.sys)
    Engine_1.1.26050.11/            mpengine.dll, mpengine_etw.dll (scan engine, from Definition Updates)
    System32/                       amsi.dll, amsiproxy.dll, smartscreen.exe, MPSSVC.dll, MpSigStub.exe
  symbols/                          symsrv-format PDB store (55 PDBs)
  SYMBOLS_REPORT.md                 per-binary symbol coverage (which got PDBs, which did not)
  README.md
```

- 115 binaries total (`.dll`/`.exe`/`.sys`). Most Defender names ship in two builds
  (inbox and updated engine) with different signatures, so they appear under both
  `binaries/` subfolders.
- `symbols/` is a standard symbol-server store keyed by each PDB's GUID + age
  (55 PDBs). Every PDB Microsoft publishes for these exact files has been fetched.
- Of 66 distinct binary names, 47 have a public PDB; 19 do not (see below).

## Source

| Item | Origin |
|------|--------|
| Inbox binaries (x64) | `C:\Program Files\Windows Defender` |
| Defender set (x86) | `C:\Program Files (x86)\Windows Defender` |
| Engine binaries | `C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.26050.15-0` |
| Kernel drivers | `C:\Windows\System32\drivers\Wd*.sys` (v4.18.25080.5) |
| Scan engine | `C:\ProgramData\Microsoft\Windows Defender\Definition Updates\...` (mpengine v1.1.26050.11) |
| System32 (AMSI/SmartScreen/Firewall) | `amsi.dll`, `amsiproxy.dll`, `smartscreen.exe`, `MPSSVC.dll`, `MpSigStub.exe` |
| Symbols | `https://msdl.microsoft.com/download/symbols` (via `symchk`) |

Captured 2026-06-26 on Windows 11 24H2. Versions are pinned in the folder names;
binaries are exact copies, unmodified.

## Symbol coverage

`symchk` was run over the whole `binaries/` tree against the public server, so the
store holds every PDB Microsoft publishes for these exact files. 47 of 66 distinct
names are covered; the remaining 19 have no public PDB and nothing more can be
downloaded for them.

Covered includes the core modules (`MsMpEng.exe`, `MpClient.dll`, `MpRtp.dll`,
`MpSvc.dll`, `MpOAV.dll`, `MpCmdRun.exe`, `NisSrv.exe`, `MpDefenderCoreService.exe`),
all 4 kernel drivers (`WdFilter`, `WdBoot`, `WdDevFlt`, `WdNisDrv`), `amsi.dll`,
`amsiproxy.dll`, `smartscreen.exe`, and `MPSSVC.dll`.

The 19 names with no public PDB fall into two groups:
- Microsoft does not publish them: the scan engine (`mpengine.dll`,
  `mpengine_etw.dll`), `MpSigStub.exe`, `MpUxAgent.dll`, the `Ds*` data-submission
  modules, and the `DefenderAiPlatform*` / `DefenderDiag` / `DefenderAgentScan` set.
- Resource-only / manifest / license DLLs that contain no code and so cannot have a
  PDB: `MsMpRes`, `MpEvMsg`, `MpAsDesc`, `EppManifest`, `MsMpLics`.

See `SYMBOLS_REPORT.md` for the full per-binary lists.

## Using the symbols

The store auto-resolves the correct PDB per build by GUID. Point your tool's
symbol path at the `symbols` folder.

WinDbg / cdb:

```
.sympath SRV*<repo>\msdefender\symbols
.reload /f
```

Ghidra: add the `symbols` folder as a PDB symbol-server / search path before
analyzing a binary from `binaries/`.

## Refreshing symbols

Requires the Windows SDK Debugging Tools (`symchk.exe`).

```powershell
& "C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\symchk.exe" `
  /r ".\binaries" `
  /s "SRV*.\symbols*https://msdl.microsoft.com/download/symbols"
```

## Notes for this repository

- Binaries and PDBs are large (about 435 MB total). If this is committed directly,
  consider Git LFS for `binaries/` and `symbols/`, or keep them out of version
  control and treat this folder as a regeneratable cache.
- An older engine build (`Platform\4.18.26040.7-0`) is also present on the source
  machine but was intentionally not copied; it is a superseded duplicate of the
  4.18.26050.15-0 set. Add it only if you need to diff builds.
- All files are Microsoft-owned and redistributable only under Microsoft's terms;
  they are kept here for local reverse-engineering reference, not redistribution.
