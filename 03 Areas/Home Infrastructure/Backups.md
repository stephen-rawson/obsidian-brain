---
type: note
updated: 2026-09-25
---

# Backups

**Rule: 3 copies, 2 media, 1 off-site.** Off-site is the remaining gap.

| Data | Primary | Copy 1 | Copy 2 |
|---|---|---|---|
| NAS shares (configs, docs, backups folder) | NAS RAID 1 | Hyper Backup → USB SSD (weekly) | *(todo: B2 / C2)* |
| Home Assistant | HA host | daily backup → NAS `ha-backups` | via Hyper Backup |
| Pi-hole + Unbound config | Pi | weekly script → NAS `backups/pihole` | via Hyper Backup |
| Pi OS image | — | monthly `dd` → NAS | via Hyper Backup |
| Router config | router | manual `.dss` → NAS `backups/router` | via Hyper Backup |
| Bitwarden vault | Bitwarden cloud | weekly CLI export → NAS `backups/bitwarden` | Proton Drive |
| Obsidian vault | laptop | Git (private repo) | LiveSync → NAS *(planned)* |
| Media library | NAS | **not backed up** (re-obtainable) | — |

## Keys and passphrases
Hyper Backup encryption, HA backup encryption, Bitwarden export passphrase,
LiveSync passphrase — **all in Bitwarden**, with the Bitwarden export passphrase
also on paper. Emergency access granted to Raya.

## Restore tests
- [x] Bitwarden export imported into a throwaway account
- [ ] HA backup restored into a test instance
- [ ] Single file restored from Hyper Backup
Untested backups are not backups. Schedule one test per quarter.

## Powershell script for Proton Drive
[Link]("C:\Users\steph\Scripts\001 Jobs\nas-to-proton.ps1")
```
# =====================================================================
# NAS -> Proton Drive off-site copy
# Daily: newest bw / router / HA backup + pi-backups mirror
# Sundays: encrypted 7z of the whole docker config share (unclean copy:
#          containers keep running; treat the Hyper Backup copy on the
#          USB SSD as the authoritative one for SQLite databases)
# Schedule: Task Scheduler, daily 02:00 + at log on, run whether
#           logged on or not, highest privileges.
# =====================================================================

$nas  = "\\192.168.1.42"
$dst  = "$env:USERPROFILE\Proton Drive\stephenrawson\My files\05 Reference\08 Home Network and Technology\999 Backups"
$date = Get-Date -Format yyyyMMdd

# 7-Zip archive password — keep the same string in Bitwarden "NAS 7zip Password")
$pw   = "LOOK ME UP"
$7z   = "C:\Program Files\7-Zip\7z.exe"

New-Item -ItemType Directory -Force -Path $dst | Out-Null

# ---------------------------------------------------------------------
# 1. newest single file from each share
# ---------------------------------------------------------------------
foreach ($p in @(
    @{ From = "$nas\bw-backups";       Filter = "*.json" },
    @{ From = "$nas\router-backups";   Filter = "*.dss"  },
    @{ From = "$nas\ha-backups\backups"; Filter = "*.tar" }
)) {
    Get-ChildItem $p.From -Filter $p.Filter -File -ErrorAction SilentlyContinue |
        Sort-Object LastWriteTime -Descending | Select-Object -First 1 |
        Copy-Item -Destination $dst -Force
}

# ---------------------------------------------------------------------
# 2. whole small shares, mirrored (Pi disk images excluded: re-creatable)
# ---------------------------------------------------------------------
robocopy "$nas\pi-backups" "$dst\pi-backups" /MIR /R:0 /W:0 /NFL /NDL /XF *.img.gz

# ---------------------------------------------------------------------
# 3. weekly encrypted archive of the docker config share
#    -mhe=on encrypts filenames as well as contents
#    exclusions drop regenerable artwork/metadata/caches
# ---------------------------------------------------------------------
if ((Get-Date).DayOfWeek -eq 'Sunday' -and (Test-Path $7z)) {
    $archive = Join-Path $dst "docker-$date.7z"

    & $7z a -t7z -mx=5 -mhe=on -p"$pw" "$archive" "$nas\docker\*" `
        -xr!Cache -xr!cache -xr!logs -xr!Logs `
        -xr!MediaCover -xr!metadata -xr!transcodes -xr!"*.log" `
        -xr!data -xr!"*.img.gz" | Out-Null

    if ($LASTEXITCODE -le 1) {
        # keep the four most recent archives
        Get-ChildItem $dst -Filter 'docker-*.7z' |
            Sort-Object Name -Descending | Select-Object -Skip 4 |
            Remove-Item -Force
    } else {
        "$(Get-Date -Format s) 7z FAILED exit $LASTEXITCODE" |
            Out-File "$dst\_last-run.txt"
        exit 1
    }
}

# ---------------------------------------------------------------------
# 4. heartbeat (Home Assistant watches this file's age)
# ---------------------------------------------------------------------
"$(Get-Date -Format s) sync ok" | Out-File "$dst\_last-run.txt"
exit 0
```

## Scheduling setup for proton backup
```
$script = "C:\Users\steph\Scripts\001 Jobs\nas-to-proton.ps1"

$action  = New-ScheduledTaskAction -Execute "powershell.exe" `
           -Argument "-NoProfile -ExecutionPolicy Bypass -File `"$script`""
$trigger = New-ScheduledTaskTrigger -Daily -At 2:00am
$settings = New-ScheduledTaskSettingsSet -StartWhenAvailable `
            -DontStopIfGoingOnBatteries -AllowStartIfOnBatteries `
            -ExecutionTimeLimit (New-TimeSpan -Hours 2)
$principal = New-ScheduledTaskPrincipal -UserId $env:USERNAME `
             -LogonType Interactive -RunLevel Highest

Register-ScheduledTask -TaskName "NAS to Proton Drive" `
    -Action $action -Trigger $trigger -Settings $settings -Principal $principal
```