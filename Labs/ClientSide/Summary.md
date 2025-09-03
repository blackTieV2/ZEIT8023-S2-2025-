### Client-Side Device Analysis Report  

A controlled forensic capture investigated traffic from an Android device connected via VLAN80. Baseline protocol analysis revealed a mix of DNS, TLS, QUIC, UDP, ICMP, and occasional HTTP flows. Focused inspection confirmed cleartext HTTP requests directed to `ip-api.com`, exposing IP and geolocation metadata without encryption. DNS queries included `allawnos.com`, an anomalous domain outside typical application dependencies.  

Encrypted sessions dominated the traffic, with TLS and QUIC flows to Google, Cloudflare, and Telegram infrastructure. Telegram activity was verified through IP ranges attributed to Russian ASN space and Cloudflare nodes, consistent with messaging app behavior. UDP broadcasts were attributed to the PortDroid application, generating LAN discovery traffic. ICMP packets were error and control frames encapsulating fragments of QUIC traffic, rather than covert channels.  

From a security assessment perspective, key concerns include the leakage of device metadata through unencrypted HTTP to third‑party providers and unexplained DNS queries. The reliance on encrypted messaging applications, particularly those hosted in foreign jurisdictions, introduces monitoring challenges and potential policy conflicts. While other traffic was consistent with expected application activity, these findings underscore the importance of enforcing encrypted communication, monitoring unusual DNS activity, and reviewing acceptable use of applications that bypass visibility boundaries.  

**Word Count: 200**
