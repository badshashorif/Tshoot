# 📘 ISP IP & Service Design Documentation

## 🔹 Device Loopback IP Plan

**Subnet:** `172.30.0.0/16`

```text
CORE & NAT Routers     → 172.30.0.0/24
ACCESS Routers        → 172.30.100.0/22
POP Switch Block-1    → 172.30.30.0/22
POP OLT Block-1    → 172.30.50.0/22
```

---

## 🔹 Access Router Service VLAN & Peering P2P IP Plan

```text
INT   VLAN: 3001–3100   → 172.25.0.0/16
CDN   VLAN: 3201–3300   → 172.26.0.0/16
FNA   VLAN: 3401–3500   → 172.27.0.0/16
GGC   VLAN: 3601–3700   → 172.28.0.0/16
BDIX  VLAN: 3801–3900   → 172.29.0.0/16
```

---

## 🔹 PPPoE Service Package List

| Package (Speed) | Profile Name      | Price (BDT) |
| --------------- | ----------------- | ----------- |
| 500TK Regular   | Extream\_Regular  | 500         |
| 30M Family      | Extream\_Family   | 600         |
| 40M Prime       | Extream\_Prime    | 700         |
| 50M Freedom     | Extream\_Freedom  | 800         |
| 60M Silver      | Extream\_Silver   | 1000        |
| 70M Gold        | Extream\_Gold     | 1200        |
| 80M Platinum    | Extream\_Platinum | 1500        |
| 100M Diamond    | Extream\_Diamond  | 2000        |

---

## 🔹 PPPoE IP Pool Allocation

```text
Expired Users     → 10.250.0.0/16
500TK Regular     → 10.11.0.0/16
600TK Family      → 10.12.0.0/16
700TK Prime       → 10.13.0.0/16
800TK Freedom     → 10.14.0.0/16
1000TK Silver     → 10.15.0.0/16
1200TK Gold       → 10.16.0.0/16
1500TK Platinum   → 10.17.0.0/16
2000TK Diamond    → 10.18.0.0/16
```

---

## 🔹 Visual Representation (Diagram Style)

```text
                   +------------------+
                   |   CORE / NAT     |
                   | Loopback 172.30.0.0/24
                   +------------------+
                             |
             --------------------------------------
             |                                    |
   +-------------------+                +-------------------+
   |  ACCESS RTR       |                |   POP SWITCHES    |
   | 172.30.100.0/22   |                | 30.0/22 & 50.0/22 |
   +-------------------+                +-------------------+

      VLAN BASED SERVICE PEERING (P2P)
      --------------------------------
      INT   → 172.25.0.0/16
      CDN   → 172.26.0.0/16
      FNA   → 172.27.0.0/16
      GGC   → 172.28.0.0/16
      BDIX  → 172.29.0.0/16

      PPPoE SERVICE POOLS
      -------------------
      Expired → 10.250.0.0/16
      Users   → 10.11.0.0/16 → 10.18.0.0/16
```

---

