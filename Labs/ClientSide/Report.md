# 🔐 Forensic Wireless Traffic Analysis Report  
**Case ID:** ZEIT8023-S2-2025  
**Analyst:** [Your Name]  
**Date:** 02 Sept 2025  

---

## 1. Executive Summary  
This investigation analyzed wireless traffic for an Android device (OnePlus 11, Android 15, IP: 192.168.80.8) within a controlled laboratory VLAN environment. Captures were passively collected via Cisco Catalyst SPAN and reviewed using tcpdump and tshark.  

Key findings:  

- Cleartext HTTP connections were established to `ip-api.com`, exposing metadata in unencrypted form.  
- DNS queries to `allawnos.com` occurred during the capture, representing anomalous name resolution.  
- Strong presence of encrypted Telegram traffic, carried by TLS and QUIC, to Cloudflare‑hosted and Russian ASN IP ranges.  
- UDP/7788 broadcasts identified as attributable to the PortDroid LAN discovery tool.  
- ICMP traffic observed was limited to error/control messaging and packet encapsulation by the network.  
- Capture did not include EAPOL frames, hence analysis covers post‑association activity only.  

Risk rating is moderate: most observed traffic correlates with expected app behavior, but unencrypted HTTP leaks and unexplained DNS queries highlight weaknesses requiring defensive attention.  

---

## 2. Scope & Objectives  
The investigation sought to:  

1. Validate reconnaissance leads from baseline traffic analysis.  
2. Attribute observed traffic flows to applications executed on the Android device.  
3. Determine whether anomalies indicated covert channels, data exfiltration, or benign app traffic.  
4. Assess overall risk posture and provide actionable recommendations.  

Analysis is limited to traffic generated after association, given the absence of EAPOL frames.  

---

## 3. Setup & Methodology  

### Network Setup  
- **ISP WAN** connected to **Firewalla Pro Gold (NAT device)**.  
- **Cisco Catalyst 3850** carried VLAN99 (mgmt) and VLAN80 (test device).  
- **Asus AP** bridged wireless clients into VLAN80.  
- **hpKali forensic laptop** received a SPAN mirror of the AP uplink port, ensuring full wireless client visibility.  
- **Android device:** OnePlus 11, Android 15, test apps installed (WhatsApp, Signal, Telegram, LinkedIn, Reddit, PortDroid, MyFirst Circle, VK).  

### Capture Procedure  
- Traffic mirrored from Gi3/0/27 (AP uplink) to Gi3/0/28 (forensic laptop).  
- Captures taken with tcpdump, stored in `.pcap` format.  
- SHA‑256 hashes used to validate file integrity (chain of custody).  

### Methodology Workflow  
1. **Baseline Recon:** Protocol hierarchy and top endpoints established.  
2. **Focused Analysis:** Extracted flows by protocol (DNS, TCP, UDP, TLS, QUIC, ICMP, HTTP).  
3. **Temporal Statistics:** Counted events per minute to identify burst vs idle activity.  
4. **Suspicious Indicators:** Queried non‑standard domains, cleartext HTTP flows, and traffic anomalies.  
5. **App Attribution:** Mapped IPs, DNS, and SNI to specific apps.  

Technical command outputs and recon artifacts are preserved in the **Appendix**.  

---

## 4. Evidence Analysis  

**Structured Evidence Table**  

| Packet No. | Protocol | Destination (IP / Host) | Verdict / Interpretation |
|------------|----------|--------------------------|--------------------------|
| 78         | HTTP     | 132.147.115.10 (`ip-api.com`) | Cleartext HTTP request—geo/IP data exposure |
| 3272       | HTTP     | 208.95.112.1 (`ip-api.com`)  | Cleartext HTTP request—unprotected |
| 45         | DNS      | Query: `allawnos.com` → 8.8.8.8 | Anomalous DNS resolution attempt |
| 3222       | TLS      | 104.16.184.241 (Cloudflare)   | Encrypted TLS session, common CDN |
| 11418      | QUIC     | 104.18.41.41 (Cloudflare)     | QUIC handshake, encrypted |
| 11165      | TLS      | 104.21.112.1 (Cloudflare)     | Encrypted TLS session |
| 8870       | TLS      | 13.215.61.135 (AWS)           | Encrypted TLS session |
| 21157      | TLS      | 13.35.33.148 (AWS/CloudFront) | Encrypted TLS session |
| 605        | TLS      | 142.251.12.95 (Google)        | Encrypted TLS session |
| 3578 / 4197| TLS      | 95.163.41.56 / 95.142.206.3 (Telegram ASN) | Telegram application traffic |
| 13459      | ICMP     | 104.18.41.41 (Cloudflare)     | ICMP control—fragment of QUIC packet |
| 10751      | ICMP     | 93.186.226.73 (Telegram infra) | ICMP control—fragment of QUIC packet |
| 3216       | ICMP     | 192.168.99.254 → 192.168.80.8 | ICMP echo/control |
| UDP/7788   | UDP      | 255.255.255.255 (broadcast)   | PortDroid discovery broadcasts |

---

## 5. Findings & Discussion  
- The device transmitted unencrypted metadata to `ip-api.com` over HTTP (risk of interception).  
- DNS activity included `allawnos.com`, outside expected application sets.  
- TLS/QUIC sessions dominated traffic, with attribution to Google, Cloudflare (likely app/CDN traffic), and Telegram.  
- Telegram use confirmed via IPs in Russian ASN space and Cloudflare front ends.  
- UDP/7788 discovery consistent with PortDroid scanning tests.  
- ICMP traffic consisted only of error/control frames, encapsulating fragments of QUIC packets.  

---

## 6. Risk Assessment  

- **High Risk:** Cleartext HTTP leaks to `ip-api.com`.  
- **Medium Risk:** DNS query to `allawnos.com` requires monitoring.  
- **Low Risk:** UDP/7788 (PortDroid scans) and ICMP error traffic are normal.  
- **Residual Risk:** Telegram encrypted flows to Russian ASN infrastructure limit visibility and may conflict with policy.  

---

## 7. Recommendations  
- Enforce HTTPS/TLS‑only outbound policies; prevent cleartext connections to services such as `ip-api.com`.  
- Implement DNS monitoring/blacklisting for anomalous domains like `allawnos.com`.  
- Enable DNS‑over‑HTTPS for better confidentiality of queries.  
- Audit and restrict application use generating discovery traffic (e.g., PortDroid).  
- Apply enterprise policy controls for use of encrypted messengers connecting to sensitive regions.  

---

## 8. Conclusion  
The forensic analysis confirmed that the test Android device generated cleartext HTTP requests, anomalous DNS queries, and extensive encrypted messaging activity via Telegram. UDP discovery and ICMP traffic were consistent with normal functions. While no exfiltration or rogue access activity was identified, risk is present due to unencrypted data leaks (`ip-api.com`) and unexplained DNS queries (`allawnos.com`). Defensive postures should enforce encryption, monitor anomaly domains, and consider policy implications of messenger applications using infrastructure in foreign ASNs.  

---

📎 **Appendices:**  
- Baseline Protocol Hierarchy  
- Top Endpoints  
- DNS Queries / Top Domains  
- TLS SNI  
- QUIC Sessions  
- Temporal Statistics  
- Conversation Tables  
- Suspicious Indicator Extracts  

---

✅ This is a professional‑grade, fact‑driven forensic report — with evidence table making it quick to reference findings.  

Would you like me to also **format this as a DOCX file** (ready to submit) with headers, table, and proper section styling?
