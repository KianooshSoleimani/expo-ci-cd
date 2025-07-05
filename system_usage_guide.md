# System Usage and Remaining Resources Guide

## Current System Status

### 📁 Disk Usage
- **Total disk space**: 512GB
- **Used**: 17GB (4%)
- **Available**: 495GB (96%)
- **Status**: ✅ Plenty of space available

### 🧠 Memory Usage
- **Total RAM**: 30GB
- **Used**: 1.9GB (6%)
- **Available**: 28GB (94%)
- **Cached**: 8.7GB
- **Status**: ✅ Excellent memory availability

### 💾 CPU Usage
- **Load average**: 2.46, 1.22, 0.48 (1min, 5min, 15min)
- **CPU idle**: 97.6%
- **CPU usage**: 2.4%
- **Status**: ✅ Very low CPU usage

### 📂 Workspace Usage
- **Workspace size**: 1.2MB
- **Status**: ✅ Minimal workspace usage

## Commands to Check Usage

### 1. Disk Usage and Space
```bash
# Check disk usage for all mounted filesystems
df -h

# Check usage of specific directory
du -sh /workspace
du -sh /home/username

# Check largest directories
du -h --max-depth=1 /workspace | sort -hr

# Check disk usage with inodes
df -i
```

### 2. Memory Usage
```bash
# Check memory usage
free -h

# More detailed memory information
cat /proc/meminfo

# Real-time memory monitoring
watch -n 1 free -h
```

### 3. CPU Usage
```bash
# Real-time CPU and process monitoring
top

# One-time snapshot
top -bn1 | head -20

# CPU usage by core
htop  # if installed

# CPU information
cat /proc/cpuinfo
lscpu
```

### 4. System Processes and Resources
```bash
# List all processes with resource usage
ps aux

# Processes sorted by CPU usage
ps aux --sort=-%cpu | head -10

# Processes sorted by memory usage
ps aux --sort=-%mem | head -10

# System resource usage
vmstat 1 5
```

### 5. Network Usage
```bash
# Network interface statistics
cat /proc/net/dev

# Network connections
netstat -tuln  # if installed
ss -tuln       # modern alternative

# Network usage monitoring (if available)
iftop  # if installed
nload  # if installed
```

### 6. I/O and Storage Performance
```bash
# Install sysstat if needed
sudo apt update && sudo apt install sysstat

# I/O statistics
iostat -x 1 5

# Disk I/O monitoring
iotop  # if installed
```

## Quick Status Check Script
You can create a quick status check script:

```bash
#!/bin/bash
echo "=== System Usage Summary ==="
echo "Date: $(date)"
echo ""
echo "--- Disk Usage ---"
df -h /
echo ""
echo "--- Memory Usage ---"
free -h
echo ""
echo "--- CPU Load ---"
uptime
echo ""
echo "--- Top 5 CPU Processes ---"
ps aux --sort=-%cpu | head -6
echo ""
echo "--- Top 5 Memory Processes ---"
ps aux --sort=-%mem | head -6
```

## Monitoring and Alerts
For continuous monitoring, you can:
1. Use `watch` command: `watch -n 5 'df -h && free -h'`
2. Set up system monitoring tools like `htop`, `glances`, or `nmon`
3. Create cron jobs to log usage periodically

## Summary
Your current system is in excellent condition with:
- ✅ 96% disk space available (495GB free)
- ✅ 94% memory available (28GB free)  
- ✅ Very low CPU usage (2.4%)
- ✅ Minimal workspace usage (1.2MB)

You have plenty of resources available for any tasks you need to perform!