# Windows Defender — binaries + symbols

Captured: 2026-06-26

## Sources
- `C:\Program Files\Windows Defender` (inbox build)
- `C:\ProgramData\Microsoft\Windows Defender\Platform\4.18.26050.15-0` (updated engine)
- Symbol server: `https://msdl.microsoft.com/download/symbols`

## Layout
```
msdefender/
  binaries/
    ProgramFiles_WindowsDefender/      40 files (inbox x64 build)
    ProgramFiles_x86_WindowsDefender/  8 files (32-bit build)
    Platform_4.18.26050.15-0/          56 files (updated build) + Drivers/ (5 .sys incl ksld.sys)
    Drivers/                           4 System32 kernel drivers .sys (v4.18.25080.5)
    Engine_1.1.26050.11/               mpengine.dll, mpengine_etw.dll
    SecurityHealth/                    Windows Security frontend (3 .exe, v10.0.26100)
    System32/                          amsi.dll, amsiproxy.dll, smartscreen.exe, MPSSVC.dll, MpSigStub.exe
  symbols/                             symsrv-format PDB store (59 PDBs)
  etw/                                 6 ETW provider manifests (.man, no symbols)
  SYMBOLS_REPORT.md
```

## Totals
- Binaries copied : **123** files (`.dll`/`.exe`/`.sys`)
- Distinct binary names : **70**
- PDBs downloaded : **59** (every PDB the public server has for these exact files)
- Names WITH public symbols : **51**
- Names WITHOUT public symbols : **19** (no public PDB exists, or resource-only DLL)

## How to use the symbols
The `symbols/` folder is a standard symsrv store, keyed by each PDB's GUID+age,
so a debugger resolves the exact PDB for the exact build automatically. Point your
tool's symbol path at it:

- **WinDbg/cdb:**  `.sympath SRV*C:\Users\Mitch\source\repos\windows_research\msdefender\symbols`
- **Ghidra:**  add the `symbols` folder as a PDB symbol-server / search path.

Because each Defender binary ships in **two builds** (inbox vs updated engine) with
different signatures, Microsoft typically publishes the PDB for one build. The
downloaded PDB matches that specific build; the debugger silently ignores the
non-matching copy. (This is why `symchk` reported 44 per-file "mismatched" lines
even though 40 distinct names are covered.)

## Binaries WITH public symbols (51 names)
AMMonitoringProvider.dll, ConfigSecurityPolicy.exe, DefenderCSP.dll, DirectML.dll,
DlpUserAgent.exe, DsCoreFull.dll, endpointdlp.dll, MdeDiag.dll,
Microsoft.Windows.AI.MachineLearning.dll, Microsoft.WindowsAppRuntime.Bootstrap.dll,
Microsoft.WindowsAppRuntime.dll, MipDlp.dll, MipDlp.exe, MpAzSubmit.dll, MpClient.dll,
MpCmdRun.exe, MpCommu.dll, MpCopyAccelerator.exe, MpDefenderCoreService.exe,
MpDetours.dll, MpDetoursCopyAccelerator.dll, MpDlp.dll, MpDlpCmd.exe, MpDlpService.exe,
mpextms.exe, MpOAV.dll, MpProvider.dll, MpRtp.dll, MpSenseComm.dll, MpSvc.dll,
MpUpdate.dll, MsMpCom.dll, MsMpEng.exe, NisSrv.exe, NpRep.dll, OfflineScannerShell.exe,
onnxruntime.dll, onnxruntime_providers_shared.dll, ProtectionManagement.dll, shellext.dll,
amsi.dll, amsiproxy.dll, smartscreen.exe, MPSSVC.dll,
WdFilter.sys, WdBoot.sys, WdDevFlt.sys, WdNisDrv.sys, ksld.sys,
SecurityHealthService.exe, SecurityHealthHost.exe, SecurityHealthSystray.exe

## Binaries WITHOUT public symbols (19 names)
mpengine.dll, mpengine_etw.dll, MpSigStub.exe,
DefenderAgentScan.exe, DefenderAiPlatform.dll, DefenderAiPlatformHost.exe,
DefenderDiag.dll, DefenderSessionHelper.exe, DsClientBase.dll, DsClientFull.dll,
DsCoreBase.dll, DsMain.dll, MdOdrMcpFilter.exe, MpUxAgent.dll,
EppManifest.dll, MpAsDesc.dll, MpEvMsg.dll, MsMpLics.dll, MsMpRes.dll
(Most are resource-only/localization DLLs — MsMpRes, MpEvMsg, MpAsDesc, EppManifest —
which carry no code and therefore no PDB.)

## Notes
- `mpengine.dll` / `mpengine_etw.dll` (the core AV scan engine, v1.1.26050.11) are
  included under `binaries/Engine_1.1.26050.11/` but have **no public symbols** —
  Microsoft does not publish engine PDBs on the public server. Copied from the
  Definition Updates folder, not the Platform dir.
- The 4 kernel drivers (`WdFilter`, `WdBoot`, `WdDevFlt`, `WdNisDrv`, v4.18.25080.5)
  all have public symbols.

## Re-run / refresh symbols later
```powershell
& "C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\symchk.exe" `
  /r "C:\Users\Mitch\source\repos\windows_research\msdefender\binaries" `
  /s "SRV*C:\Users\Mitch\source\repos\windows_research\msdefender\symbols*https://msdl.microsoft.com/download/symbols"
```
