## Running commands
- One time: `powershell -ExecutionPolicy Bypass -File .\script.ps1`
- In the folder: `Get-ChildItem "C:\Users\steph\Scripts" -Recurse -Filter *.ps1 | Unblock-File`

## Running scripts (Lenovo, PowerShell)
- One-off, without changing policy: `powershell -ExecutionPolicy Bypass -File .\script.ps1`
- Standing setting (once): `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`
- Unblock a single downloaded script: `Unblock-File .\script.ps1`
  - Avoid `-Recurse` across the whole Scripts folder; it unblocks anything that lands there.