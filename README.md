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
    System32/                       amsi.dll (Antimalware Scan Interface)
  symbols/                          symsrv-format PDB store (43 PDBs)
  SYMBOLS_REPORT.md                 per-binary symbol coverage (which got PDBs, which did not)
  README.md
```

- 97 binaries total (`.dll`/`.exe`). Most Defender names ship in two builds (inbox
  and updated engine) with different signatures, so they appear under both
  `binaries/` subfolders.
- `symbols/` is a standard symbol-server store keyed by each PDB's GUID + age.

## Source

| Item | Origin |
|------|--------|
| Inbox binaries | `C:\Program Files\Windows Defender` |
| Engine binaries | `C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.26050.15-0` |
| amsi.dll | `C:\Windows\System32\amsi.dll` (v10.0.26100.7309) |
| Symbols | `https://msdl.microsoft.com/download/symbols` (via `symchk`) |

Captured 2026-06-26 on Windows 11 24H2. Versions are pinned in the folder names;
binaries are exact copies, unmodified.

## Symbol coverage

- 40 of 56 Defender binary names have public symbols, including the core modules
  (`MsMpEng.exe`, `MpClient.dll`, `MpRtp.dll`, `MpSvc.dll`, `MpOAV.dll`,
  `MpCmdRun.exe`, `NisSrv.exe`, `MpDefenderCoreService.exe`) plus `amsi.dll`.
- 16 names have no public PDB; most are resource-only or localization DLLs that
  contain no code (`MsMpRes`, `MpEvMsg`, `MpAsDesc`, `EppManifest`).
- `mpengine.dll` (the core scan engine) is not included: it ships inside the
  signature/VDM package, not the Platform directory, and its symbols are not public.

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

- Binaries and PDBs are large (about 377 MB total). If this is committed directly,
  consider Git LFS for `binaries/` and `symbols/`, or keep them out of version
  control and treat this folder as a regeneratable cache.
- All files are Microsoft-owned and redistributable only under Microsoft's terms;
  they are kept here for local reverse-engineering reference, not redistribution.
