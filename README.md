# ============================================
# System Health Check Script
# Author: Muad Muhammed
# Purpose: Quick health check for Windows servers
# ============================================

Write-Host "========================================" -ForegroundColor Cyan
Write-Host "   SYSTEM HEALTH CHECK REPORT" -ForegroundColor Cyan  
Write-Host "   $(Get-Date)" -ForegroundColor Cyan
Write-Host "========================================" -ForegroundColor Cyan

# --- CPU Check ---
Write-Host "
[CPU]" -ForegroundColor Yellow
$cpu = Get-WmiObject Win32_Processor | Select-Object -ExpandProperty LoadPercentage
Write-Host "CPU Usage: $cpu%"
if ($cpu -gt 80) {
    Write-Host "WARNING: CPU usage is HIGH" -ForegroundColor Red
} else {
    Write-Host "Status: OK" -ForegroundColor Green
}

# --- Memory Check ---
Write-Host "
[MEMORY]" -ForegroundColor Yellow
$mem = Get-WmiObject Win32_OperatingSystem
$totalMem = [math]::Round($mem.TotalVisibleMemorySize / 1MB, 2)
$freeMem  = [math]::Round($mem.FreePhysicalMemory  / 1MB, 2)
$usedMem  = [math]::Round($totalMem - $freeMem, 2)
Write-Host "Total RAM : $totalMem GB"
Write-Host "Used  RAM : $usedMem GB"
Write-Host "Free  RAM : $freeMem GB"

# --- Disk Check ---
Write-Host "
[DISK]" -ForegroundColor Yellow
$disks = Get-PSDrive -PSProvider FileSystem
foreach ($disk in $disks) {
    if ($disk.Used -ne $null) {
        $total = [math]::Round(($disk.Used + $disk.Free) / 1GB, 1)
        $used  = [math]::Round($disk.Used / 1GB, 1)
        $free  = [math]::Round($disk.Free / 1GB, 1)
        Write-Host "Drive $($disk.Name): Total=${total}GB  Used=${used}GB  Free=${free}GB"
        if ($free -lt 10) {
            Write-Host "  WARNING: Low disk space on drive $($disk.Name)!" -ForegroundColor Red
        }
    }
}

# --- Key Services Check ---
Write-Host "
[SERVICES]" -ForegroundColor Yellow
$services = @("wuauserv", "WinDefend", "Spooler")
foreach ($svc in $services) {
    $s = Get-Service -Name $svc -ErrorAction SilentlyContinue
    if ($s) {
        $color = if ($s.Status -eq "Running") { "Green" } else { "Red" }
        Write-Host "$($s.DisplayName): $($s.Status)" -ForegroundColor $color
    }
}

Write-Host "
========================================"
Write-Host "Check complete. Review warnings above."
Write-Host "========================================"
