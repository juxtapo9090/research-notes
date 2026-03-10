# FAFO: Terminal Blindness - Node.exe Output Invisible

**Date:** 2026-01-14
**Status:** ✅ SOLVED

## Problem
- `node --version` and `npm --version` produce **empty output** through Antigravity terminal
- Same commands work fine in standalone PowerShell (admin)
- Other commands (`python --version`, `where.exe`, `Write-Host`) work fine

## Root Cause 🎯
**Windows Compatibility Flag: RUNASADMIN**

Registry key:
```
HKCU:\Software\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Layers
C:\NPM\node.exe : ~ RUNASADMIN
```

This flag forces node.exe to require elevation. When running in non-admin terminal (like Antigravity's integrated terminal), it **fails silently** instead of showing an error.

## Solution
Remove the compatibility flag:
```powershell
Remove-ItemProperty -Path "HKCU:\Software\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Layers" -Name "C:\NPM\node.exe"
```

## Verification
After fix:
```
node --version → v25.3.0 ✅
npm --version  → 11.7.0 ✅
```

## How This Flag Got Set
Likely set via:
1. Right-click node.exe → Properties → Compatibility → "Run as administrator"
2. Or Windows auto-applied it after a failed elevation prompt

## Lesson Learned
When a command works in admin PowerShell but produces **empty output** (not error) in non-admin terminal, check:
```powershell
Get-ItemProperty "HKCU:\Software\Microsoft\Windows NT\CurrentVersion\AppCompatFlags\Layers" -Name "C:\path\to\exe"
```
