# 👻 Ghost Home Node

![Home Assistant](https://img.shields.io/badge/Home%20Assistant-41BDF5?style=for-the-badge&logo=home-assistant&logoColor=white)
![Linux](https://img.shields.io/badge/Linux%20Mint-87CF3E?style=for-the-badge&logo=linux-mint&logoColor=white)
![KVM](https://img.shields.io/badge/KVM%2FQEMU-FF6600?style=for-the-badge&logo=qemu&logoColor=white)
![Privacy](https://img.shields.io/badge/Privacy%20First-000000?style=for-the-badge&logo=tor-project&logoColor=white)
![No Cloud](https://img.shields.io/badge/No%20Cloud-DC382D?style=for-the-badge&logo=icloud&logoColor=white)

---

## Manifesto

> **"Your home. Your data. Your rules."**
>
> Ghost Home Node is a sovereign domotics infrastructure built on recycled enterprise hardware. Zero cloud dependencies. Zero telemetry. Complete operational security. This is home automation for those who understand that convenience should never come at the cost of privacy.

---

## Hardware Stack

| Component | Model | Rationale |
|-----------|-------|-----------|
| **Host Server** | Lenovo ThinkPad T480 | Enterprise-grade reliability. Dual-battery system acts as **integrated UPS** — no external power backup required. Easily serviceable and upgradeable. |
| **Host OS** | Linux Mint | Stable Debian base, excellent hardware support, no telemetry. |
| **Robot Vacuum** | Dreame L10 Pro | **LIDAR-equipped** for precision mapping. Compatible with [Valetudo](https://valetudo.cloud/) for cloud extraction. |
| **Camera** | Reolink E1 Zoom | RTSP stream support. **Local-only operation** — no cloud account required. |
| **Zigbee Coordinator** | Sonoff Dongle-P (CC2652P) | Texas Instruments chipset. Proven compatibility with Zigbee2MQTT. Future-proof for Thread/Matter. |
| **Storage** | SSD/HDD (NO SD Cards) | SD cards (especially generic "Bliksem" types) are a **ticking time bomb** for 24/7 operations. SSD or bust. |

---

## Installation Guide

### Phase 1: Prepare the Host

**1.1 — Install Virtualization Stack**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

**1.2 — Add User to Libvirt Groups**

```bash
sudo usermod -aG libvirt,kvm $USER
newgrp libvirt
```

**1.3 — Verify KVM Support**

```bash
kvm-ok
# Expected: INFO: /dev/kvm exists — KVM acceleration can be used
```

**1.4 — Download Home Assistant OS**

Get the **Latest Stable** KVM image (`.qcow2`) from the official releases:

```bash
wget https://github.com/home-assistant/operating-system/releases/download/<VERSION>/haos_ova-<VERSION>.qcow2.xz
xz -d haos_ova-<VERSION>.qcow2.xz
```

> ⚠️ **Critical:** Never use Release Candidate (`rc`) versions for production. They will break.

---

### Phase 2: Deploy the Virtual Machine

**2.1 — Launch virt-manager**

```bash
virt-manager
```

**2.2 — Create New VM**

- **Connection:** QEMU/KVM
- **Installation:** Import existing disk image
- **Disk:** Select your `.qcow2` file
- **OS Type:** Generic Linux 2022
- **Memory:** `4096 MB` minimum (8192 recommended)
- **CPUs:** `2` minimum (4 recommended)

**2.3 — Configure VM Settings (Before First Boot)**

Open VM configuration → **Customize before install**:

| Setting | Value |
|---------|-------|
| **Firmware** | `UEFI x86_64: /usr/share/OVMF/OVMF_CODE.fd` |
| **Chipset** | Q35 |
| **Network** | NAT (virbr0) |

> 💡 **Why NAT?** Bridge mode on Wi-Fi hosts is notoriously problematic. NAT provides stable connectivity without kernel-level workarounds.

---

## ⚠️ Pitfalls & Troubleshooting

This section documents the **actual problems encountered** during deployment. Read this before you debug for hours.

### Pitfall #1: VM Won't Boot (Black Screen)

**Symptom:** VM starts but immediately halts or shows no display.

**Root Cause:** Home Assistant OS requires UEFI firmware.

**Solution:**

1. In virt-manager, go to VM settings → **Overview** → **Firmware**
2. Change from `BIOS` to `UEFI x86_64: /usr/share/OVMF/OVMF_CODE.fd`
3. Delete and recreate the VM if firmware change is grayed out

---

### Pitfall #2: "Access Denied" on Boot

**Symptom:** UEFI shell displays `Access Denied` error.

**Root Cause:** Secure Boot is enabled but HAOS isn't signed.

**Solution:**

1. During VM boot, press `ESC` or `DEL` to enter UEFI settings
2. Navigate to **Device Manager** → **Secure Boot Configuration**
3. Set **Secure Boot** to `Disabled`
4. Save and exit (F10)

```
┌────────────────────────────────────────┐
│        Secure Boot Configuration       │
│────────────────────────────────────────│
│  Current Secure Boot State: [Enabled]  │
│  Secure Boot Mode:          [Standard] │
│                                        │
│  > Attempt Secure Boot      [✗]        │  ← Disable this
└────────────────────────────────────────┘
```

---

### Pitfall #3: No Network / Can't Reach HAOS

**Symptom:** VM boots but `homeassistant.local:8123` is unreachable.

**Root Cause:** `virbr0` NAT network isn't active or DHCP failed.

**Solution:**

```bash
# Check network status
virsh net-list --all

# If 'default' is inactive:
virsh net-start default

# Nuclear option — reset the entire NAT network:
virsh net-destroy default
virsh net-start default
```

Then reboot the VM. Check assigned IP:

```bash
virsh net-dhcp-leases default
```

Access HAOS at `http://<IP_ADDRESS>:8123`

---

## Post-Installation

### Enable Advanced Mode

`Settings` → `User Profile` → Toggle **Advanced Mode**

This unlocks access to system-level configurations.

---

### Install Terminal & SSH Add-on

1. Go to **Settings** → **Add-ons** → **Add-on Store**
2. Search and install: **Terminal & SSH**
3. Configure SSH access:

```yaml
# Add-on Configuration
authorized_keys:
  - ssh-rsa AAAA... your-public-key
password: ""  # Leave empty if using key-based auth
```

4. Start the add-on and enable **Start on boot**

---

### Install HACS (Home Assistant Community Store)

SSH into Home Assistant, then:

```bash
wget -O - https://get.hacs.xyz | bash -
```

Restart Home Assistant:

`Settings` → `System` → `Restart`

After reboot: `Settings` → `Devices & Services` → `Add Integration` → Search "HACS"

---

### Install Mosquitto MQTT Broker

1. **Settings** → **Add-ons** → **Add-on Store**
2. Install **Mosquitto broker**
3. Configure credentials:

```yaml
logins:
  - username: mqtt_user
    password: <STRONG_PASSWORD>
```

4. Start and enable **Start on boot**

The MQTT broker is now available at `mqtt://homeassistant.local:1883`

---

## Roadmap

- [ ] **Zigbee Integration** — Pair Sonoff Dongle-P via Zigbee2MQTT
- [ ] **Robot Liberation** — Root Dreame L10 Pro with Valetudo (bye-bye Xiaomi cloud)
- [ ] **Camera Integration** — RTSP stream into Frigate NVR
- [ ] **Network Hardening** — Implement VLANs for IoT isolation
- [ ] **Firewall Rules** — UFW/iptables lockdown on host
- [ ] **Backup Strategy** — Automated HAOS snapshots to encrypted external storage
- [ ] **Monitoring** — Prometheus + Grafana stack for infrastructure metrics

---

## Philosophy

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│   "A smart home that phones home isn't smart — it's a      │
│    surveillance device you pay electricity for."            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

This project exists because:

- **Cloud services die.** Companies pivot, servers shut down, APIs break. Your home shouldn't stop working because a startup ran out of runway.
- **Data is leverage.** Your movement patterns, sleep schedules, and presence data are worth money. Keep them off corporate servers.
- **Resilience matters.** When the internet goes down, your lights should still work.

---

## License

MIT — Do whatever you want. Just don't blame me when you're 4 hours deep into UEFI debugging at 2 AM.

---

<p align="center">
  <i>Built with paranoia and recycled ThinkPads.</i>
</p>
