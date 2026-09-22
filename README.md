# 🔵 BT-Hijack v1.1.0

**Bluetooth Security Assessment Tool — Educational Use Only**  
**أداة تقييم أمني للبلوتوث — للاستخدام التعليمي فقط**

---

## ⚠️ Legal Disclaimer / إخلاء المسؤولية القانونية

> **English:** This tool is developed **for educational and authorized security research purposes ONLY**.  
> Using this tool against any device **without explicit written permission** from the owner is a **criminal offense** under most jurisdictions (Computer Fraud and Abuse Act, UK Computer Misuse Act, etc.).  
> The developer assumes **no liability** for misuse.

> **العربية:** هذه الأداة مُطوَّرة للأغراض **التعليمية والبحثية المرخّصة فقط**.  
> استخدامها على أجهزة دون تصريح مكتوب يُعدّ **جريمة جنائية**. المطوِّر غير مسؤول عن أي سوء استخدام.

---

## 💡 Overview / نظرة عامة

**BT-Hijack** is an educational tool that demonstrates the full Bluetooth attack chain, based on **CVE-2023-45866** (Bluetooth HID Injection vulnerability).

It shows how an attacker can:
1. **Scan** — discover nearby Classic & BLE devices
2. **Deauthenticate** — force-disconnect a device from its paired host
3. **Spoof** — clone the MAC address and device name of a trusted device
4. **Hijack** — impersonate a Bluetooth keyboard (HID) to inject keystrokes
5. **Report** — generate an HTML/JSON security assessment report

---

## 🧩 Architecture / بنية المشروع

```
BT-Hijack/
├── main.py                  # Entry point — CLI arguments & interactive mode
├── requirements.txt         # Python dependencies
├── setup_kali.sh            # Automated setup script for Kali Linux
│
├── modules/
│   ├── scan.py              # Bluetooth scanner (Classic + BLE via bleak)
│   ├── spoof.py             # MAC & name spoofing via hciconfig + gdbus
│   ├── hijack.py            # HID injection engine (CVE-2023-45866)
│   ├── dos.py               # L2CAP deauthentication flood
│   ├── interactive.py       # Interactive menu (Full Attack Chain)
│   ├── connection_map.py    # Displays paired device relationships
│   └── report.py            # HTML + JSON report generator
│
└── utils/
    ├── adapter.py           # Bluetooth adapter detection & selection
    ├── platform_check.py    # Runtime capability detection per OS
    ├── config.py            # Centralized configuration loader
    ├── logger.py            # Structured logging
    └── errors.py            # Custom exception hierarchy
```

---

## 🔬 CVE-2023-45866 — The Core Vulnerability

**CVE-2023-45866** is a Bluetooth security vulnerability affecting Android, iOS, and Linux systems.  
It allows an **unauthenticated** Bluetooth device to pair as a **HID Keyboard** without user confirmation.

### Attack Flow:
```
[Attacker — Kali Linux]
       │
       │ 1. Scan: find victim phone + trusted headset
       │ 2. L2CAP flood → force headset to disconnect from phone
       │ 3. Spoof: clone headset MAC + name on Kali adapter
       │ 4. Configure Kali as HID Keyboard (class=0x002540)
       │ 5. Victim phone auto-reconnects → connects to Kali instead
       │ 6. Inject keystrokes into victim phone
       ▼
[Victim Phone]
```

### Why it works:
- Bluetooth auto-reconnect is based on **MAC address only**
- Phone trusts the device because it matches the saved MAC + name
- CVE-2023-45866 allows pairing as keyboard **without PIN**

---

## 📦 Requirements / المتطلبات

### System Requirements
| Component | Requirement |
|-----------|------------|
| OS | **Kali Linux** (recommended) or any Linux with BlueZ |
| Python | 3.8 or higher (tested on 3.11 – 3.14) |
| Bluetooth | USB or built-in adapter with HCI support |
| Privileges | **Must run as root** (`sudo`) |

### System Tools (BlueZ — pre-installed on Kali)
```bash
hciconfig       # MAC address spoofing
bluetoothctl    # Device management & name spoofing
hcitool         # Connection info & device scanning
btmgmt          # Device class configuration (HID mode)
l2ping          # L2CAP connectivity testing & flood
gdbus           # D-Bus interface for name changes
```

### Python Libraries
| Library | Purpose | Install |
|---------|---------|---------|
| `bleak` | BLE scanning | `pip install bleak` |
| `colorama` | Colored terminal output | `pip install colorama` |
| `PyYAML` | Config file parsing | `pip install PyYAML` |
| `pydbus` | BlueZ D-Bus disconnect API | `sudo apt install python3-pydbus` |
| `pybluez2` | L2CAP raw sockets (DoS only) | `sudo apt install python3-bluez` |

> **Note:** `pybluez2` is **only required** for the L2CAP DoS feature.  
> All other features (scan, spoof, HID hijack) work without it.

---

## 🔧 Installation / التثبيت

### Quick Setup on Kali Linux (Recommended)

```bash
# 1. Clone or copy the project
cd ~/Downloads/BT/BT

# 2. Install system Bluetooth tools
sudo apt update
sudo apt install bluez bluez-tools python3-pydbus python3-bluez -y

# 3. Install Python libraries
sudo pip3 install bleak colorama PyYAML

# 4. Verify setup
sudo python3 main.py --capabilities
```

### Or use the automated setup script:
```bash
chmod +x setup_kali.sh
sudo ./setup_kali.sh
```

---

## 🚀 Usage / طريقة الاستخدام

### Interactive Mode (Recommended for demo)
```bash
sudo python3 main.py --interactive
```

### Command-Line Mode
```bash
# Scan for devices
sudo python3 main.py --scan --timeout 15

# Full attack chain against a target
sudo python3 main.py --target AA:BB:CC:DD:EE:FF --attack full

# Generate report only
sudo python3 main.py --report
```

### Interactive Menu Options
```
[1] Scan & Select Target        — discover nearby Bluetooth devices
[2] View Connected Pairs        — show known device relationships
[3] Full Attack Chain           — automated: scan → deauth → spoof → hijack
[4] Generate Report             — create HTML/JSON security report
[5] Disconnect / Unpair Device  — force-disconnect a specific device
[0] Exit
```

---

## 🎯 Full Attack Chain — Step by Step

### Prerequisites
- Victim phone and target headset must be **powered on and nearby**
- Kali must have a working Bluetooth adapter (`hciconfig hci0 up`)
- Run as **root** (`sudo`)

### Steps in the Tool

```
Step 1/4: Scan
  → Select victim phone as Target
  → (scan must show the phone's MAC)

Step 2/4: Spoof Target Selection
  → Select headset (the device paired to the phone) as Spoof Target

Step 3/4: Disconnect
  → Tool sends L2CAP flood to headset → headset drops connection from phone

Step 4/4: Spoof + HID Hijack
  → Tool clones headset MAC + name on hci0
  → Configures hci0 as HID Keyboard (class=0x002540)
  → Phone auto-reconnects → connects to Kali
  → Keystrokes injected into phone
```

---

## ✅ Feature Capability Matrix

| Feature | Linux (Kali) | Windows | Requires |
|---------|-------------|---------|----------|
| BLE Scan | ✅ | ✅ | `bleak` |
| Classic Scan | ✅ | ⚠️ limited | `python3-bluez` or `bluetoothctl` |
| MAC Spoof | ✅ | ❌ | `hciconfig` (BlueZ) |
| Name Spoof | ✅ | ❌ | `bluetoothctl` / `gdbus` |
| L2CAP DoS | ✅ | ❌ | `python3-bluez` |
| HID Hijack | ✅ | ❌ | BlueZ + root |
| Report | ✅ | ✅ | built-in |

---

## 📊 Sample Output

```
  Platform : Linux
  Feature          Status
  ------------------------------------
  ble_scan         [OK]
  classic_scan     [OK]
  dos              [OK]
  deauth           [OK]
  spoof_mac        [OK]
  spoof_name       [OK]
  hijack           [OK]
  report           [OK]
```

---

## 🔒 Defense Recommendations / توصيات الحماية

1. **Disable Bluetooth when not in use** — أطفئ البلوتوث عند عدم الاستخدام
2. **Keep OS updated** — CVE-2023-45866 is patched in Android Dec 2023+, iOS 16.6+
3. **Use Secure Simple Pairing** — Disable legacy pairing modes
4. **Monitor unexpected HID connections** — Treat unknown keyboard connections as suspicious
5. **Use BLE instead of Classic BT** — BLE has stronger pairing security

---

## 📝 License

For educational and research purposes only.  
**DO NOT** use against devices you do not own or have explicit permission to test.
