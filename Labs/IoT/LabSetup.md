


```mermaid
%%{init: { "flowchart": { "htmlLabels": false } }}%%
flowchart TD
    Internet((Internet / ISP))
    Cloud["Tuya Cloud\nAWS / Cloudflare / Google"]
    Router["Firewalla Gold Pro (Router)\n192.168.99.254\nNAT outbound"]
    Switch["Cisco Catalyst C3850\nSVI VLAN99: 192.168.99.1\nSVI VLAN80: 192.168.80.1 (Gateway)"]
    AP["ASUS RT-AC3100\nAP Mode → VLAN80 Bridge"]
    Laptop["Kali Laptop (192.168.80.11)\nDocker: Home Assistant + SPAN Capture"]
    Camera["Tuya Wi-Fi Camera\n192.168.80.12\nShenzhen Bilian Electronic Co."]

    %% Infra path
    Internet <--> Router
    Router <--> Switch
    Switch <--> AP
    Switch <--> Laptop
    AP <--> Camera

    %% Camera outbound flows
    Camera -->|"TLS / QUIC sessions"| Switch
    Switch --> Router
    Router --> Internet
    Internet --> Cloud

    %% Mgmt / SPAN path
    Laptop -.->|"Mgmt via HA\nSPAN monitoring"| Camera

```

