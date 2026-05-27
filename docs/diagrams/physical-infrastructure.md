# Physical Infrastructure Layout

```mermaid
graph TB
    subgraph "Datacenter Environment"
        subgraph "Rack/Location"
            subgraph "Proxmox Host: lab2"
                LAB2HW["Intel Xeon W-2123<br/>4 cores, 8 threads @ 3.60GHz<br/>32 GB RAM<br/>94 GB System SSD<br/>916 GB Data Disk"]
                
                subgraph "VMs on lab2"
                    CP1VM[k3s-cp-01<br/>VMID 105<br/>4GB, 40GB]
                    CP2VM[k3s-cp-02<br/>VMID 108<br/>4GB, 40GB]
                    CP3VM[k3s-cp-03<br/>VMID 107<br/>4GB, 50GB]
                    WK1VM[k3s-wk-01<br/>VMID 109<br/>4GB, 100GB]
                    WK2VM[k3s-wk-02<br/>VMID 111<br/>6GB, 90GB]
                    WK3VM[k3s-wk-03<br/>VMID 112<br/>4GB, 88.5GB]
                    LBVM[k3s-lb-01<br/>VMID 119<br/>4GB, 13.5GB]
                end
            end

            subgraph "Proxmox Host: steven"
                STEVENHW["AMD A10-7850K<br/>4 cores, 4 threads<br/>11 GB RAM<br/>94 GB System SSD"]
                
                subgraph "VMs on steven"
                    DBVM[FooballPoolDB01<br/>VMID 102<br/>4GB, 60GB<br/>PostgreSQL]
                    ANSIBLEVM[ansible-control<br/>VMID 104<br/>4GB, 40GB]
                end
            end

            NETWORK[Network Switch<br/>10.0.0.0/24]
            POWER[PDU / Power]
        end

        INTERNET[Internet Connection]
    end

    %% Physical connections
    LAB2HW -.runs.-> CP1VM & CP2VM & CP3VM
    LAB2HW -.runs.-> WK1VM & WK2VM & WK3VM
    LAB2HW -.runs.-> LBVM
    STEVENHW -.runs.-> DBVM & ANSIBLEVM
    
    LAB2HW ---|10 GbE NIC| NETWORK
    STEVENHW ---|1 GbE NIC| NETWORK
    NETWORK ---|Firewall/Router| INTERNET
    
    LAB2HW & STEVENHW -.powered by.-> POWER

    %% Styling
    classDef hardware fill:#ffebee,stroke:#c62828,stroke-width:3px
    classDef vm fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    classDef infrastructure fill:#fff3e0,stroke:#e65100,stroke-width:2px

    class LAB2HW,STEVENHW hardware
    class CP1VM,CP2VM,CP3VM,WK1VM,WK2VM,WK3VM,LBVM,DBVM,ANSIBLEVM vm
    class NETWORK,POWER,INTERNET infrastructure
```

## Physical Resource Summary

### Datacenter Resources

**Total Physical Hosts:** 2  
**Total CPU Cores:** 8 physical (12 logical threads)  
**Total RAM:** 43 GB  
**Total Storage:** ~1.1 TB  
**Network:** 10.0.0.0/24 private network

---

## Host Specifications

### Primary Host: "lab2"

**Hardware:**
- **Model:** Workstation-class server
- **CPU:** Intel Xeon W-2123
  - Architecture: x86_64
  - Cores: 4
  - Threads: 8 (Hyper-Threading enabled)
  - Base Clock: 3.60 GHz
  - Cache: L1: 256 KB, L2: 4 MB, L3: 8.25 MB (shared)
- **RAM:** 32 GB DDR4
- **Storage:**
  - Disk 1: 94 GB SSD (Proxmox system)
  - Disk 2: 916 GB (dedicated for Immich photos)
- **Network:** 
  - Interface: enp0s31f6
  - Speed: 1 Gbps (capable of 10 Gbps)
- **IP Address:** 10.0.0.232

**Resource Utilization:**
- RAM Usage: 27 GB / 32 GB (84% - mostly k8s VMs)
- CPU Usage: ~95% average
- Storage: 5% used on system disk

**VMs Hosted:** 8 total (7 running, 1 stopped)
- All 6 Kubernetes cluster nodes
- 1 Load balancer
- 1 Template VM (stopped)

**Role:** Primary production Kubernetes infrastructure

---

### Secondary Host: "steven"

**Hardware:**
- **Model:** Desktop/Workstation (repurposed)
- **CPU:** AMD A10-7850K Radeon R7
  - Architecture: x86_64
  - Cores: 4
  - Threads: 4 (no SMT)
  - Base Clock: 3.7 GHz
  - APU: 12 compute cores (4 CPU + 8 GPU)
- **RAM:** 11 GB DDR3
- **Storage:**
  - Disk 1: 94 GB SSD (Proxmox system)
- **Network:**
  - Interface: enp2s0
  - Speed: 1 Gbps
- **IP Address:** 10.0.0.237

**Resource Utilization:**
- RAM Usage: 7.9 GB / 11 GB (72%)
- CPU Usage: ~99% average
- Storage: 26% used on system disk
- Swap: 2.2 GB / 8 GB in use

**VMs Hosted:** 8 total (2 running, 6 stopped)
- 1 Production database
- 1 Ansible automation controller
- 6 Stopped VMs (templates, old VMs)

**Role:** Support services (database, automation)

---

## Network Infrastructure

### Physical Network

```
Internet
   ↓
Firewall/Router
   ↓
Network Switch (10.0.0.0/24)
   ├── lab2 (10.0.0.232)
   ├── steven (10.0.0.237)
   ├── Pi-hole DNS
   └── Other devices
```

**Network Configuration:**
- **Subnet:** 10.0.0.0/24 (254 usable IPs)
- **Gateway:** 10.0.0.1 (assumed)
- **DNS:** Pi-hole (internal) + upstream resolvers
- **DHCP:** Static IPs for servers, dynamic for other devices

### Virtual Networking

**Proxmox Bridges:**
- `vmbr0` on both hosts
- Bridged to physical NICs
- All VMs connected to vmbr0
- Full layer-2 connectivity between hosts

**Kubernetes Networking:**
- **Node Network:** 10.0.0.0/24 (physical)
- **Pod Network:** 10.42.0.0/16 (CNI overlay)
- **Service Network:** 10.43.0.0/16 (ClusterIP)
- **Ingress VIP:** 10.0.0.119 (MetalLB)

---

## Power & Cooling

**Power Distribution:**
- PDU or datacenter power (assumed)
- UPS backup (assumed in datacenter)
- Estimated power consumption:
  - lab2: ~150-200W under load
  - steven: ~100-150W under load

**Cooling:**
- Datacenter HVAC
- Individual CPU coolers on each host
- No custom cooling required

---

## Storage Architecture

### lab2 Storage

```
Physical Disks:
├── /dev/sda (94 GB SSD)
│   ├── /dev/sda1 - BIOS boot
│   ├── /dev/sda2 - EFI (1 GB)
│   └── /dev/sda3 - LVM
│       ├── pve-root (Proxmox system)
│       └── pve-data (VM images)
└── /dev/sdb (916 GB)
    └── /dev/sdb1 - /mnt/immich-storage
```

**Storage Allocation:**
- Proxmox System: ~4 GB used / 94 GB
- VM Images: ~420 GB allocated
- Immich Storage: 2 MB used / 916 GB available

### steven Storage

```
Physical Disks:
└── /dev/sda (94 GB SSD)
    ├── /dev/sda1 - BIOS boot
    ├── /dev/sda2 - EFI (1 GB)
    └── /dev/sda3 - LVM
        ├── pve-root (Proxmox system)
        └── pve-data (VM images)
```

**Storage Allocation:**
- Proxmox System: ~23 GB used / 94 GB (26%)
- VM Images: ~100 GB allocated

---

## Virtualization Layer

### Proxmox VE Configuration

**Version:** 9.0 (both hosts)
- Kernel: 6.14.x
- KVM/QEMU virtualization
- LVM storage backend
- Cloud-init template support

**Features Used:**
- VM snapshots
- Live migration (between compatible nodes)
- Resource monitoring
- Backup capabilities
- Web UI management

### VM Provisioning

**Automation:**
- Terraform creates VMs from cloud-init templates
- Ansible configures OS and installs software
- Standardized VM templates (Ubuntu 24.04, Rocky 9)

**Template VMs:**
- ubuntu-24.04-cloudinit (VMID 9000, 9100)
- rocky-9-cloudinit (VMID 9001)

---

## Disaster Recovery Considerations

### Single Points of Failure

**Identified:**
1. ✅ All k8s nodes on single physical host (lab2)
2. ✅ No host-level redundancy
3. ✅ Single network switch (assumed)
4. ⚠️ Database on separate host (actually good!)

**Mitigations:**
- Kubernetes provides application-level HA
- Velero backups to external storage
- Can manually migrate VMs if needed
- Database isolated from k8s cluster failures

### Backup Strategy

**VM-Level:**
- Proxmox backup capabilities
- External backup target (planned)

**Application-Level:**
- Velero for Kubernetes resources
- Longhorn snapshots for volumes
- Database dumps for PostgreSQL

---

## Scalability Path

### Vertical Scaling (Per Host)

**lab2 Upgrade Options:**
- RAM: 32 GB → 64 GB or 128 GB
- CPU: Already enterprise-grade
- Storage: Add more data disks

**steven Upgrade Options:**
- RAM: 11 GB → 16 GB or 32 GB
- CPU: Limited by socket
- Storage: Add more disks

### Horizontal Scaling

**Add More Hosts:**
- Purchase additional Proxmox host
- Migrate some k8s nodes for true HA
- Set up Proxmox cluster
- Implement Ceph distributed storage

**Add More VMs:**
- Current hosts can support 2-3 more VMs each
- lab2 has RAM headroom (5 GB free)
- steven is more constrained (2.7 GB free)

---

## Cost Considerations

**Datacenter Hosting:**
- Power costs
- Cooling costs
- Network bandwidth
- Physical space rental
- Remote hands if needed

**Benefits of Current Setup:**
- Repurposed hardware (lower capital cost)
- Right-sized for workload
- Room for growth
- Professional environment
