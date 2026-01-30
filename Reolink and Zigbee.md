# 📡 SENSORS.md — Phase 2: Sensory Integration

> **Ghost Home Project** | Technical Documentation  
> *Air-Gapped / Privacy-First Architecture*

---

## 🎯 Objective

Deploy sensory modules (Sight & Hearing) while ensuring **complete network isolation** from Cloud services. No data leaves the local perimeter.

---

## 👁️ Vision Module: "Ghost Mode" Camera

### Hardware

| Component | Specification |
|-----------|---------------|
| Model | **Reolink E1 Zoom** |
| Firmware | Recent version (update recommended before isolation) |
| Connectivity | Wi-Fi 2.4/5 GHz, WPA2 |

### Problem Statement

Reolink firmware imposes two major constraints:

1. **Web Interface Disabled** — HTTP/HTTPS ports are blocked by default, forcing use of the proprietary mobile app
2. **Cloud Dependency** — Without intervention, the camera communicates with Reolink servers
3. **Limited ISP Router** — No option to block internet access by MAC address

### 🛡️ Solution: "Ghost Gateway" Procedure

#### Step 1 — Controlled Initialization

Temporary connection via Reolink mobile app:

```
Network     : Main Wi-Fi (WPA2)
Application : Reolink (iOS/Android)
Mode        : Initial setup
```

#### Step 2 — Protocol Unlocking

In the mobile app, navigate to advanced settings:

```
Settings > Advanced Settings > Port Settings
```

Manually enable the following protocols:

| Protocol | Port | Purpose |
|----------|------|---------|
| RTSP | 554 | Real-time video stream |
| RTMP | 1935 | Streaming |
| ONVIF | 8000 | Standardized discovery & control |
| HTTPS | 443 | Secure Web interface |

#### Step 3 — "Ghost Gateway" Isolation

> ⚠️ **WARNING: CRITICAL STEP**  
> This manipulation permanently cuts Cloud access. The camera will only remain accessible on the local network. Any reconfiguration will require a factory reset.

Static network configuration with **intentionally invalid gateway**:

```
IP Mode     : Static
IP Address  : 192.168.1.X        # Fixed IP on LAN
Subnet Mask : 255.255.255.0
Gateway     : 192.168.1.222      # ← FAKE GATEWAY (router = .1)
DNS 1       : 0.0.0.0            # or 127.0.0.1
DNS 2       : 0.0.0.0
IPv6        : DISABLED
```

**Hack Principle**: By declaring a non-existent gateway, all traffic destined to leave the LAN (→ Internet) is routed into the void. The camera remains fully functional locally.

#### Step 4 — Home Assistant Integration

```yaml
# Configuration via official Reolink integration
Integration : Reolink
Method      : Local IP (192.168.1.X)
Protocol    : RTSP/ONVIF
```

### Validation

| Test | Expected Result |
|------|-----------------|
| Ping from LAN | ✅ Response OK |
| RTSP stream in HA | ✅ Real-time video |
| Connection via 4G (off-LAN) | ❌ Failure confirmed |
| Reolink app (off-LAN) | ❌ "Camera offline" |

**Status: OPERATIONAL — Isolation validated.**

---

## ⚡ Sensor Module: Zigbee Infrastructure

### Hardware

| Component | Specification |
|-----------|---------------|
| Model | **Sonoff Zigbee 3.0 USB Dongle Plus** |
| Variant | **Model P** (Texas Instruments) |
| Chipset | CC2652P |
| Interface | USB — CP210x UART Bridge |

### Prerequisite: KVM/QEMU Virtualization

The USB dongle plugged into the Linux host is not visible by default in the Home Assistant VM.

#### USB Passthrough Configuration

```
Application : virt-manager
VM          : Home Assistant OS

Navigation:
  VM Details > Add Hardware > USB Host Device

Selection:
  Device: "CP210x UART Bridge" / "ITead Sonoff Zigbee"
```

After adding, restart the VM to apply changes.

### Software Configuration: ZHA

Using the native **Zigbee Home Automation (ZHA)** integration.

> ⚠️ **WARNING: RADIO TYPE SELECTION**  
> Radio protocol selection is **irreversible without resetting** the Zigbee network. An error here will corrupt the configuration.

#### Radio Compatibility Table

| Dongle Model | Chipset | ZHA Radio Type |
|--------------|---------|----------------|
| Sonoff **P** | CC2652P (Texas Instruments) | **ZNP** ✅ |
| Sonoff **E** | EFR32MG21 (Silicon Labs) | EZSP |

```
ZHA Configuration:

Radio Type       : ZNP (Texas Instruments Z-Stack)
Serial Port      : /dev/ttyUSB0 (or auto-detected)
Baud Rate        : 115200
Flow Control     : Hardware

Zigbee Network   : New network
Channel          : 15 (default)
```

### Validation

| Test | Expected Result |
|------|-----------------|
| Dongle detection in HA | ✅ `/dev/ttyUSB0` visible |
| ZHA initialization | ✅ Network created (Channel 15) |
| Test sensor pairing | ✅ Discovery < 60s |

**Status: OPERATIONAL — Zigbee mesh active.**

---

## 🛡️ OpSec Summary

### Isolation Matrix

| Module | LAN Access | WAN Access | Vendor Cloud |
|--------|------------|------------|--------------|
| Reolink E1 Zoom | ✅ | ❌ Blocked | ❌ None |
| Zigbee (ZHA) | ✅ | N/A | ❌ None |
| Home Assistant | ✅ | Controlled | ❌ None |

### Applied Principles

1. **Zero Trust Network** — Every IoT device is considered hostile by default
2. **Logical Air-Gap** — Isolation via network configuration (Ghost Gateway)
3. **Open Protocols** — RTSP, ONVIF, Zigbee = non-proprietary standards
4. **Local Data** — No video/sensor stream transits through the Internet

---

## 📋 Phase 2 Checklist

- [x] Camera initialized via mobile app
- [x] RTSP/ONVIF/HTTPS ports enabled
- [x] Ghost Gateway configured (fake gateway)
- [x] WAN isolation validated (4G test)
- [x] Reolink integration in Home Assistant
- [x] USB Passthrough configured (virt-manager)
- [x] Zigbee dongle detected by VM
- [x] ZHA configured (ZNP / Channel 15)
- [x] Zigbee network operational

---

*Documentation generated for Ghost Home Project*  
*Data Sovereignty — Total Control — Zero Cloud*
