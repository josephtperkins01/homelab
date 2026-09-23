# Hardware Specifications - Linux Home Lab

These are the resource assumptions and expansion notes I used to size the VirtualBox VM. They are planning values, not claims about hardware that is not recorded elsewhere.


## Host Machine (Windows PC)

### Processor
- **CPU:** Intel Core i7 / Ryzen 7 (or equivalent)
- **Cores:** Minimum 4 cores recommended
- **Speed:** 3.0+ GHz base frequency
- **Virtualization:** VT-x/AMD-V enabled in BIOS

### Memory (RAM)
- **Total:** 16 GB minimum (32 GB recommended)
- **Allocation to VM:** 4-8 GB
- **Remaining for Host:** 8+ GB

### Storage
- **Total:** 500 GB SSD or larger
- **Free Space:** 100+ GB minimum
- **VM Image Location:** C:\Users\[Username]\VirtualBox VMs\
- **VM Disk Size:** 40-60 GB (recommended)

### Network
- **Network Adapter:** Ethernet or Wi-Fi
- **Speed:** 1 Gbps recommended
- **Connection Type:** Broadband

### Operating System
- **OS:** Windows 10 or Windows 11 (Pro or Home)
- **Build:** Latest updates installed
- **Virtualization:** Hyper-V or VirtualBox (not both enabled simultaneously)

---

## Virtual Machine (Ubuntu Server) Specifications

### VM Configuration

```
┌─────────────────────────────────────────┐
│  Ubuntu Server 24.04.4 LTS VM           │
├─────────────────────────────────────────┤
│ vCPU Allocation:           2-4 cores    │
│ RAM Allocation:            4-8 GB       │
│ Primary Disk:              40-60 GB     │
│ Disk Type:                 VDI (dynamic)│
│ Network Interface:         1 (NAT)      │
│ Display:                   Headless     │
│ Clipboard:                 Bidirectional│
│ Drag & Drop:               Disabled     │
└─────────────────────────────────────────┘
```

### CPU Configuration
- **Type:** 64-bit Linux (Ubuntu)
- **Cores:** 2 (can be increased if host allows)
- **Execution Cap:** 100% (can be limited for host stability)
- **Nested Paging:** Enabled (if supported)

### Memory Configuration
- **Base Memory:** 4096 MB (4 GB) default
- **Maximum:** 8192 MB (8 GB) for heavier workloads
- **Page Fusion:** Disabled (unless running multiple VMs)

### Storage Configuration

#### Primary Disk (System)
- **Type:** VDI (VirtualBox Disk Image)
- **Format:** Dynamic allocation (grows with usage)
- **Size:** 40-60 GB
- **Location:** `C:\Users\[Username]\VirtualBox VMs\[VM_Name]\`
- **File:** `[VM_Name].vdi`

#### Future Expansion Options
- **Secondary Disk:** Optional for separation of data
- **USB Drive:** Can be attached for backups
- **Shared Folder:** For file exchange with host

### Network Configuration

#### Adapter 1 (Primary)
- **Type:** PCNet-FAST III (or Intel PRO/1000)
- **Attached to:** NAT
- **MAC Address:** Auto-generated
- **Port Forwarding:**
  - SSH: Host 2222 → Guest 22
  - HTTP (future): Host 8080 → Guest 80
  - HTTPS (future): Host 8443 → Guest 443

#### (Optional) Adapter 2
- **Type:** Host-only (for VM-to-VM communication)
- **Attached to:** VirtualBox Host-Only Network

### Display Configuration
- **Mode:** Headless (accessible only via SSH)
- **Video Memory:** Not applicable
- **Acceleration:** Not applicable
- **Remote Desktop:** Not enabled (use SSH instead)

### USB Configuration
- **USB Controller:** Enabled (for future needs)
- **USB 2.0 Filter:** Optional

---

## Performance Expectations

### CPU Performance
- **Single Core:** ~2000-3000 Passmark score (depends on host)
- **Multi-Core:** Scales with VM core allocation
- **Expected Use:** Light to moderate workloads

### Memory Performance
- **Available RAM in VM:** 3.8-7.8 GB (after OS overhead)
- **Swap Space:** 2 GB (created during installation)
- **Expected Use:** Comfortable for web server + monitoring

### Disk I/O Performance
- **Disk Speed:** Depends on host storage
- **VM Disk:** Dynamic allocation (grows on demand)
- **Read/Write:** ~100-300 MB/s (typical for VM)
- **Expected Use:** Good for development; adequate for small production

### Network Performance
- **Throughput:** Up to 1 Gbps (NAT limited)
- **Latency:** <1 ms to host
- **Expected Use:** Sufficient for learning and small deployments

---

## Resource Monitoring

### Check VM Resource Usage (from Ubuntu)

```bash
# CPU usage
top -bn1 | head -20

# Memory usage
free -h
ps aux --sort=-%mem

# Disk usage
df -h
du -sh ~/*

# Network statistics
ip -s link
netstat -i

# Full system info
lsb_release -a
uname -a
lscpu
```

### Adjust VM Resources (from VirtualBox)

1. **Increase vCPU:**
   - Shutdown VM
   - Settings → System → Processor
   - Increase Cores (host dependent)
   - Start VM

2. **Increase RAM:**
   - Shutdown VM
   - Settings → System → Motherboard
   - Increase Base Memory
   - Start VM

3. **Increase Disk Space:**
   - Settings → Storage → Select disk
   - Modify disk size (extends live)
   - From Ubuntu: `sudo lvextend -l +100%FREE /dev/ubuntu-vg/ubuntu-lv` and `sudo resize2fs /dev/ubuntu-vg/ubuntu-lv`

---

## Backup Strategy

### VM Backup Locations
- **VDI File:** `C:\Users\[Username]\VirtualBox VMs\[VM_Name]\[VM_Name].vdi`
- **Snapshots:** `C:\Users\[Username]\VirtualBox VMs\[VM_Name]\Snapshots\`
- **Backup Frequency:** Weekly (recommended)

### Backup Methods

#### Method 1: VirtualBox Snapshots
```bash
# From Windows Command Line
VBoxManage snapshot "[VM_Name]" take "[Snapshot_Name]" --description "[Description]"
```

#### Method 2: File Copy
```powershell
# Copy VDI file to external backup
Copy-Item "C:\Users\[Username]\VirtualBox VMs\[VM_Name]\[VM_Name].vdi" "D:\Backup\homelab_backup.vdi"
```

#### Method 3: Clone VM
- Use VirtualBox Export function
- Create full VM backup for disaster recovery

---

## Expansion Possibilities

### Horizontal Scaling (Future)
- Add 2nd VM for separation of concerns
- Add 3rd VM for high-availability testing
- Network them together via Host-Only Adapter

### Vertical Scaling (Near-term)
- Increase host RAM to 32+ GB
- Allocate 8 GB to primary VM
- Add secondary disk for data separation

### Storage Expansion
- Add external USB drive for backups
- Attach secondary VDI disk to VM
- Increase primary VDI size

---

## Compatibility Notes

### Supported on Windows
- ✅ Windows 10 Pro, Home, Enterprise
- ✅ Windows 11 Pro, Home, Enterprise
- ⚠️ Requires VirtualBox 6.1+ or newer
- ⚠️ Hyper-V must be disabled if using VirtualBox

### Known Limitations
- NAT mode limits bidirectional network access (use port forwarding)
- Headless operation (no GUI — access via SSH only)
- Performance varies based on host specifications

### Performance Tips
1. Keep host free of resource-heavy applications during lab work
2. Disable Windows Background apps for VirtualBox machine
3. Use SSD for host (not HDD) for better I/O
4. Close other VMs when heavy workloads are running
5. Monitor host CPU/RAM usage with Task Manager

---

## Quick Reference: Resource Allocation Table

| Component | Minimum | Recommended | Comfortable |
|-----------|---------|-------------|------------|
| Host vCPU | 4 cores | 6+ cores | 8+ cores |
| Host RAM | 16 GB | 32 GB | 64 GB |
| VM vCPU | 1-2 cores | 2 cores | 4 cores |
| VM RAM | 2 GB | 4 GB | 8 GB |
| VM Disk | 20 GB | 40 GB | 60+ GB |

---

**Last Updated:** August 25, 2026
**VM Version:** 24.04.4 LTS
**VirtualBox Minimum:** 6.1
