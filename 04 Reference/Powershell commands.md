---
type: note
updated: 2026-10-03
aliases:
  - powershell
  - pwsh
---

# PowerShell commands

Lenovo laptop (Windows), PowerShell. General Windows reference. Anything
specific to one service belongs in that service's note under
[[Home Infrastructure]].

## Which PowerShell
| | Windows PowerShell | PowerShell 7 |
|---|---|---|
| Launch | `powershell` | `pwsh` |
| Check version | `$PSVersionTable.PSVersion` | same |
| Use for | Defaults, built-in scheduled tasks | Everything new |

## Running scripts
Scripts live in `C:\Users\steph\Scripts`.

| Task | Command |
|---|---|
| See the effective policy | `Get-ExecutionPolicy -List` |
| Standing setting (once) | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` |
| One-off, without changing policy | `powershell -ExecutionPolicy Bypass -File .\script.ps1` |
| Unblock one downloaded script | `Unblock-File .\script.ps1` |
| Is a file blocked? | `Get-Item .\script.ps1 -Stream Zone.Identifier` (error = not blocked) |

## Scheduled tasks
| Task | Command |
|---|---|
| Find tasks | `Get-ScheduledTask -TaskName "*Proton*"` |
| Run now | `Start-ScheduledTask -TaskName "<name>"` |
| Disable (before deleting) | `Disable-ScheduledTask -TaskName "<name>"` |
| Delete (after a quiet period) | `Unregister-ScheduledTask -TaskName "<name>"` |

Last run and result:
```powershell
Get-ScheduledTask -TaskName "*Proton*" | Get-ScheduledTaskInfo |
  Select-Object TaskName, LastRunTime, LastTaskResult, NextRunTime
```

`LastTaskResult`: `0` success · `267009` still running · `267011` never run ·
anything else is the script's exit code.

## Network checks
| Task | Command | Good result |
|---|---|---|
| Adapter, IP, gateway, DNS | `Get-NetIPConfiguration` | IP `192.168.1.x`, gateway `.1` |
| Name via Pi-hole | `Resolve-DnsName nas.home.stephenrawson.uk -Server 192.168.1.31` | An answer |
| Port open (SMB) | `Test-NetConnection 192.168.1.42 -Port 445` | `TcpTestSucceeded : True` |
| Flush DNS cache | `Clear-DnsClientCache` | silent |
| Tailscale | `tailscale status` | peers listed, this machine online |

## Shares and SSH
| Task | Command |
|---|---|
| List mapped drives | `Get-SmbMapping` |
| Map a share (prompts for the password) | `net use P: \\192.168.1.42\paperless-consume /user:<dsm-user> /persistent:yes` |
| Remove a mapping | `net use P: /delete` |
| SSH to servers | `ssh mini` · `ssh stephen-nas@192.168.1.42` |

## Files and logs
| Task | Command |
|---|---|
| Tail a log live | `Get-Content .\file.log -Tail 50 -Wait` |
| Search text in files | `Select-String -Path .\*.log -Pattern "error"` |
| Hash a file | `Get-FileHash .\file -Algorithm SHA256` |

Folder size in GB:
```powershell
(Get-ChildItem .\folder -Recurse -File | Measure-Object Length -Sum).Sum / 1GB
```

## Gotchas
- **`Bypass` only lasts for that one process.** It doesn't change the policy.
- **Don't run `Unblock-File` recursively over the Scripts folder.** That also
  unblocks anything that lands there later and gets run.
- **`curl` in Windows PowerShell is `Invoke-WebRequest`.** Use `curl.exe` for
  real curl.
- **Disable a scheduled task before you delete it.** Deleting loses its trigger
  and action, so there's no way back if something turns out to depend on it.
- **Laptop VPN captures DNS.** Disconnect it before trusting any DNS test
  (see [[Runbook]]).
- **A scheduled task set to "Run only when user is logged on"** silently doesn't
  run after a reboot until you log in.
- **Never put passwords in commands or scripts.** Let `net use` prompt, or use
  Credential Manager. Values live in Bitwarden.
- **Pipes inside table cells break Markdown tables.** Pipelines go in code
  blocks, not tables.

## To do
- [ ] ~9 Oct: disable the "NAS to Proton Drive" task (see [[Off-site backup (Proton Drive)]])
- [ ] Re-map `P:` with a standard DSM user, not `stephen-nas` (see [[Paperless]])

## Related
[[Home Infrastructure]] · [[Runbook]] · [[Tailscale]] · [[Paperless]] ·
[[Off-site backup (Proton Drive)]]
