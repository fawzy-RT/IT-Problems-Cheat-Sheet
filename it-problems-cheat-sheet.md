# IT Problems Cheat Sheet

Quick reference for common IT support problems and solutions

---

## Universal Troubleshooting Methodology
always remember this 
1. **Identify** — What's broken? Who's affected? When did it start?
2. **Isolate** — One device or many? Local or network-wide?
3. **Test** — Run basic diagnostics.
4. **Resolve** — Apply the simplest fix first.
5. **Verify** — Test that the original problem is gone.
6. **Document** — Write down what worked.

---

## 1. No Internet (Single PC)

**Symptoms**: User reports "no internet" but LAN is fine. Browser shows DNS errors.

**Possible Causes**:
- Network cable / adapter disabled
- DHCP failed (APIPA address 169.254.x.x)
- DNS misconfiguration
- VPN/proxy interference

**Diagnosis**:
```cmd
ipconfig /all
ping <gateway>
ping 8.8.8.8
nslookup google.com
```

![ipconfig /all output example](screenshots/no-internet-ipconfig.png)

**Solution**:
- IP starts with `169.254` → DHCP failed → `ipconfig /release` then `ipconfig /renew`
- Gateway reachable, external fails → ISP/router issue
- IP works, DNS fails → `ipconfig /flushdns` or change DNS to 8.8.8.8 / 1.1.1.1
- VPN/proxy → disable and test

---

## 2. No Internet (Multiple Users)

**Symptoms**: Several users report issues at the same time.

**Possible Causes**:
- Router/gateway failure
- Switch/AP outage
- DHCP/DNS service down
- ISP outage

**Solution**:
- Identify scope (one floor? building? everyone?)
- Check router and switch status lights
- Test gateway from authorized device
- Escalate to network team — don't make changes without authorization

---

## 3. Connected to Wi-Fi but No Internet

**Symptoms**: Windows shows "connected" but apps don't load.

**Possible Causes**:
- Captive portal needs sign-in
- Wrong SSID (guest network)
- DHCP/gateway issue
- Weak signal

**Solution**:
- Confirm SSID is correct (not guest network)
- Open browser — captive portal may load
- Run `ipconfig /all` + `ping <gateway>` + `nslookup example.com`
- Reconnect to network
- Move closer to AP if signal is weak

---

## 4. Slow PC

**Symptoms**: Slow startup, laggy apps, freezing.

**Possible Causes**:
- High CPU/memory/disk usage
- Too many startup apps
- Disk almost full
- Background updates/AV scans
- Failing hardware (especially HDD)

**Diagnosis**:
```cmd
taskmgr                    :: Ctrl+Shift+Esc — check CPU/Memory/Disk
perfmon /rel               :: Reliability Monitor — see crashes/installs timeline
```

![Task Manager showing CPU, Memory, and Disk usage](screenshots/slow-pc-task-manager.png)

**Solution**:
- 90%+ resource usage → kill or update that process
- Disable unnecessary startup apps in Task Manager → Startup tab
- Below 10% free on C: → clean up
- HDD slow → upgrade to SSD
- Pending updates → restart
- Overheating/hardware → escalate

⚠️ Don't kill processes you don't recognize — some look weird but are critical Windows services.

---

## 5. Windows Update Fails

**Symptoms**: Updates stuck, fail with error code, won't install.

**Possible Causes**:
- Low disk space
- Wrong date/time
- Pending restart
- Corrupted update components
- Company policy (WSUS/Intune)

**Solution**:
- Check disk space + date/time
- Restart if pending
- Try Windows Update troubleshooter (Settings → Troubleshoot)
- Note error code, search Microsoft Learn
- Reset Windows Update components (last resort):

```cmd
net stop wuauserv
net stop cryptSvc
net stop bits
net stop msiserver
ren C:\Windows\SoftwareDistribution SoftwareDistribution.old
ren C:\Windows\System32\catroot2 catroot2.old
net start wuauserv
net start cryptSvc
net start bits
net start msiserver
```

⚠️ Don't run scripts from random blogs — official Microsoft steps are safer.

---

## 6. Blue Screen (BSOD)

**Symptoms**: PC crashes with blue screen, shows stop code.

**Common Stop Codes**:
- `IRQL_NOT_LESS_OR_EQUAL` → usually driver issue
- `INACCESSIBLE_BOOT_DEVICE` → storage/storage driver
- `CRITICAL_PROCESS_DIED` → multiple causes
- `MEMORY_MANAGEMENT` → RAM or driver

**Solution**:
- Write down the stop code
- Note when it happens (boot? after login? during specific app?)
- Disconnect non-essential USB devices (especially docks)
- Boot to Safe Mode if possible
- Roll back recent drivers/updates
- Run `sfc /scannow` and `DISM /RestoreHealth`
- Run memory diagnostic: `mdsched.exe`
- If BitLocker is on → have recovery key ready before changes

---

## 7. Windows Won't Boot

**Symptoms**: PC won't start, restarts constantly, drops to WinRE.

**Solution**:
- Write down exact error message
- Disconnect all USB peripherals (docks, printers, external drives)
- Boot to WinRE (hold Shift+Restart)
- Try Startup Repair first
- Try Safe Mode
- Try Uninstall Updates (if recent update caused it)
- Try System Restore (if restore point exists)
- Last resort: reinstall Windows (after backing up data)

⚠️ Check BitLocker recovery key BEFORE touching boot config.

---

## 8. Printer Shows Offline

**Symptoms**: User can't print, printer shows as offline in Windows.

**Possible Causes**:
- Printer out of paper/toner
- Cable/network disconnected
- Printer IP changed (DHCP)
- Wrong port config in Windows
- Print Spooler stuck
- Driver issue

**Diagnosis**:
```cmd
ping <printer-ip>            :: Network printer reachability
services.msc                 :: Check Print Spooler status
```

**Solution**:
- Check printer physically (power, paper, toner, display for errors)
- For network printers: confirm IP matches Windows port
- Restart Print Spooler service
- Cancel stuck jobs in print queue
- Reinstall driver from manufacturer (NOT third-party sites)
- For shared printers: don't change IP without coordinating with admin

---

## 9. Print Job Stuck in Queue

**Symptoms**: Print job stuck with status "Printing", "Pending", or "Error".

**Solution**:
- Check printer for errors (paper jam, low toner)
- Open print queue → cancel stuck job
- Restart Print Spooler: `services.msc` → Print Spooler → Restart
- If spool folder corrupted, clear it (follow approved procedure only):

```cmd
net stop spooler
del /Q /F /S "%systemroot%\System32\Spool\Printers\*.*"
net start spooler
```

![Print queue with a stuck job](screenshots/print-job-stuck-queue.png)

- Reinstall driver if recurring

⚠️ Don't manually delete spool files on shared print servers — affects other users.

---

## 10. Application Crashes

**Symptoms**: App closes by itself, freezes, throws errors.

**Diagnosis**:
```cmd
perfmon /rel               :: Reliability Monitor — see crash timeline
eventvwr.msc               :: Event Viewer → Windows Logs → Application
```

![Event Viewer showing an application error](screenshots/application-crash-event-viewer.png)

**Solution**:
- Note: which app, what action caused crash, error message
- Try to reproduce
- Update the app
- Repair installation (Control Panel → Programs → Repair)
- Reset app settings (back up first)
- Reinstall per vendor instructions
- Collect logs and escalate to app vendor if recurring

⚠️ Don't delete app data folders without checking — users keep weird stuff there.

---

## 11. Device Driver Problem

**Symptoms**: Device missing from Device Manager, yellow warning, or doesn't work.

**Common Error Codes**:
- Code 28 → driver not installed
- Code 43 → device reported a problem (often driver, sometimes hardware)
- Code 10 → device can't start
- Code 31 → driver not working

**Solution**:
```cmd
devmgmt.msc                :: Device Manager
```

![Device Manager](screenshots/device-driver-problem.png)

- Right-click device → Properties → check status + error code
- Update driver from Windows Update or manufacturer website
- Roll back driver if recent update broke it
- For hardware ID: Properties → Details → Hardware Ids
- Search hardware ID online for driver

⚠️ Don't install drivers from random "driver updater" sites — often bundled with adware.

---

## 12. IP Address Conflict

**Symptoms**: Windows reports IP conflict, device keeps losing network.

**Possible Causes**:
- Two devices with same static IP
- Static IP overlapping DHCP scope
- Bad DHCP reservation

**Solution**:
```cmd
ipconfig /all
getmac /v
```
- Identify both conflicting devices (MAC + IP)
- Check DHCP reservations on router/server
- Fix duplicate assignment
- Don't randomly change IPs — you'll create new conflicts

---

## 13. User Can't Log In (Active Directory)

**Symptoms**: User reports they can't log in to domain PC.

**Possible Causes**:
- Account disabled
- Account locked (too many bad passwords)
- Password expired
- Computer account broken (broken trust)
- DNS issue can't reach DC

**Solution**:
- Check AD user account status:

```powershell
Get-ADUser -Identity jdoe | Select-Object Enabled, LockedOut, PasswordExpired
Unlock-ADAccount -Identity jdoe
Set-ADAccountPassword -Identity jdoe -Reset
```

- If computer account broken: rejoin domain
- Check DNS on client points to DC

---

## 14. Outlook Won't Connect

**Symptoms**: User can't send/receive email, Outlook shows "Disconnected".

**Possible Causes**:
- Network issue
- Cached credentials expired
- Outlook profile corrupted
- Exchange/M365 service down

**Solution**:
- Check internet connection
- Check Outlook status bar for errors
- Try Outlook Safe Mode: `outlook /safe`
- Try other user on same PC, or same user on other PC
- Recreate Outlook profile (Control Panel → Mail)
- Check M365 service health if cloud

---

## 15. Wi-Fi Network Not Showing

**Symptoms**: Computer can't find expected Wi-Fi network.

**Possible Causes**:
- Wi-Fi disabled / Airplane mode on
- Adapter driver issue
- Unsupported band (5GHz only adapter, etc.)
- Weak signal
- AP/SSID config issue

**Solution**:
- Confirm Wi-Fi is enabled + Airplane mode off
- Check if other devices see the SSID
- Check Device Manager for wireless adapter status
- Update or roll back wireless driver
- Move closer to AP and refresh network list

---

## 16. VPN Won't Connect

**Symptoms**: VPN client fails to connect or disconnects immediately.

**Possible Causes**:
- Internet issue
- VPN credentials expired
- VPN server down
- Local firewall blocking
- Cert expired

**Solution**:
- Verify internet works without VPN
- Try different network (mobile hotspot)
- Re-enter credentials
- Try different VPN protocol/server
- Check firewall isn't blocking VPN ports
- Contact VPN admin for cert/cert expiry

---

## Quick Reference — Common Ports

| Port | Service |
|------|---------|
| 22 | SSH |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 445 | SMB |
| 3389 | RDP |
---

## Quick Reference — Key CMD Commands

```cmd
ipconfig /all               :: Full IP info
ping <host>                 :: Test reachability
tracert <host>              :: Trace route
nslookup <name>             :: DNS lookup
netstat -ano                :: Connections + PIDs
sfc /scannow                :: Repair system files
```

---

## Important Notes

- Always test commands before running them in production.
- Back up data before destructive fixes.
- Document what worked for future reference.
- When in doubt, escalate don't make things worse.
