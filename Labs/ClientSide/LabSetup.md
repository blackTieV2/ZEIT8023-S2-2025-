# Setup & Methodology  

### Network Environment  
- **ISP WAN** → Firewalla Pro Gold (Router & NAT device, connected to ISP uplink)  
- **Firewalla Pro Gold** ↔ **Cisco Catalyst 3850 (L3 Switch)**  
  - Uplink **Gi3/0/12** connected to Firewalla (configured as **802.1Q trunk, VLAN99 native**, VLAN80 carried).  
- **Asus Access Point (AP)** connected to Cisco **Gi3/0/27**, bridging wireless clients into **VLAN80** (192.168.80.0/24).  
- **hpKali Laptop (Forensics Station)** connected to Cisco **Gi3/0/28**, designated as **SPAN/mirror destination** from the AP port.  

---

🌐 **Internet/ISP**
   │
   ▼
🛡 **Firewalla Gold Pro** (`192.168.99.254/24`)
   │ VLAN99 trunk
   ▼
🔀 **Cisco Catalyst 3850 (L3 Switch)**
   - VLAN99 SVI: `192.168.99.1`
   - VLAN80 SVI: `192.168.80.1`
   - SPAN: Gi3/0/27 → Gi3/0/28
   │
   ├── 📡 **ASUS RT‑AC3100 (Access Point/Bridge)** → VLAN80  
   │       - WiFi clients, Android DUT bridged  
   │
   └── 💻 **Kali Linux Forensics Station (hpKali)**  
           - NIC: `eth0` (SPAN dest)  
           - Tools: tcpdump / tshark  
           - Storage: `/root/ZEIT8023/`  
 
📱 **DUT: OnePlus 11 (Model CPH2451, Android 15)**  
- IP: `192.168.80.8`  
- Security Patch: *June 1, 2025 (Kernel 5.15.149)*  
- Apps: WhatsApp, Signal, Telegram, LinkedIn, Reddit, VK, PortDroid, MyFirst Circle

---
```mermaid
flowchart TD
    ISP[Internet / ISP]
    FW[Firewalla Gold Pro<br/>(192.168.99.254/24)]
    SW[Cisco Catalyst C3850 (L3)<br/>SVI VLAN99 = 192.168.99.1<br/>SVI VLAN80 = 192.168.80.1]
    AP[Access Point<br/>Gi3/0/27]
    KALI[Kali Monitor<br/>Gi3/0/28 (SPAN)]
    ASUS[ASUS RT-AC3100<br/>(AP Mode → VLAN80 Bridge)]
    WIFI[Wi-Fi Clients (VLAN80)]
    OP11[OnePlus 11 (CPH2451)<br/>Android 15<br/>IP: 192.168.80.8<br/>Apps: WA, Signal, Telegram, LinkedIn, Reddit, PortDroid, MyFirst Circle, VK]

    ISP --> FW
    FW --> SW
    SW --> AP
    SW --> KALI
    SW --> ASUS
    ASUS --> WIFI
    WIFI --> OP11
```


---
                

### VLAN Configuration  
- **VLAN 99** (192.168.99.0/24):  
  - Cisco SVI: `192.168.99.1`  
  - Firewalla: `192.168.99.254`  
  - Purpose: Management / router inter‑VLAN gateway.  

- **VLAN 80** (192.168.80.0/24):  
  - Cisco SVI: `192.168.80.1` (default gateway for wireless clients)  
  - NAT provided by Firewalla (static route for VLAN80).  
  - Purpose: Wireless Android device under test.  

### Cisco Switch (WS-C3850)  
- **SPAN Configuration:**  
  - **Source:** Gi3/0/27 (Asus AP uplink port)  
  - **Destination:** Gi3/0/28 (Kali forensic laptop)  
  - Mode: Monitor session (RX/TX mirrored)  
- **Routing:** Enabled (`ip routing`), with default route via `192.168.99.254`.  
- **NAT:** Handled by Firewalla (Cisco Catalyst does not support NAT in this image).  

### Firewalla Pro Gold  
- Running Linux‑based firmware.  
- Configured static route: `192.168.80.0/24 → 192.168.99.1`.  
- NAT rules confirmed via `iptables` to masquerade 192.168.80.0/24 out ISP uplink.  
- Verification: bidirectional ICMP between Cisco SVI (99.1) and Firewalla (99.254).  

### Access Point (Asus)  
- Mode: Bridge/AP only.  
- Connects wireless clients directly into VLAN80.  
- DHCP allocation provided via Cisco SVI/DHCP pool (gateway/dns: 192.168.80.1).  

### Forensic Capture Station (hpKali Laptop)  
- Host: HP laptop with **Kali Linux (2025.2 rolling)**.  
- NIC: `eth0` configured for reception of SPAN traffic.  
- Tools:  
  - **tcpdump 4.x** → raw capture to `.pcap`  
  - **tshark (Wireshark CLI 4.4.7)** → analysis extraction  
- Storage: Captures stored under `/root/ZEIT8023/`.  
- Example active capture command:  
  ```bash
  sudo tcpdump -i eth0 -nn -s0 -w redteam_ap_capture_<timestamp>.pcap
  ```  

### Device Under Test (Android Phone)  
- Connected via Wi‑Fi AP → VLAN80.  
- Received IP via DHCP: **192.168.80.8**.  
- Test Apps Installed and executed:  
  - WhatsApp  
  - Signal  
  - Telegram  
  - LinkedIn  
  - Reddit  
  - PortDroid (scanner test app)  
  - MyFirst Circle  
  - VK (vk.com, Russian social media)  

---

📌 **Chain of Custody**  
All captures were generated passively (SPAN port, no injection). Files are preserved in original `.pcap` format with SHA‑256 checksums, and working copies processed via **tshark** into `.txt` forensic artifacts.  

```bash
sha256sum redteam_ap_capture_2025-09-01_23-59.pcap dns_only_2025-09-02_00-00.pcap
```

9ceac810064e01135cb727d50230fc6e16eaa44c7dfd2687e37d55a3d98abee6  redteam_ap_capture_2025-09-01_23-59.pcap
0099ca8a749c89d792d77c95f94168f3d11e1f6111e1b1703e61a340912858a2  dns_only_2025-09-02_00-00.pcap

---

