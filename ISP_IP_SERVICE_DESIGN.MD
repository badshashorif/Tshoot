# 📘 ISP IP & Service Design Documentation

## 🔹 Device Loopback IP Plan

**Subnet:** `172.30.0.0/16`

```text
CORE & NAT Routers     → 172.30.0.0/24
ACCESS Routers         → 172.30.100.0/22
POP Switch Block-1     → 172.30.30.0/22
POP OLT Block-1        → 172.30.50.0/22
SERVER MGMG BLOCK      → 172.30.172.0/22
```

---

## 🔹 Retail Access Router & Corporate Router Service VLAN & Peering P2P IP Plan

```text
Retail ACCESS RTR:
INT   VLAN: 3101–3200   → 172.25.0.0/17
GGC   VLAN: 3201–3300   → 172.26.0.0/17
FNA   VLAN: 3301–3400   → 172.27.0.0/17
BDIX  VLAN: 3401–3500   → 172.28.0.0/17
CDN   VLAN: 3501–3600   → 172.29.0.0/17

Corporate POP RTR:
INT   VLAN: 3601–3700   → 172.25.128.0/17
GGC   VLAN: 3701–3800   → 172.26.128.0/17
FNA   VLAN: 3801–3900   → 172.27.128.0/17
BDIX  VLAN: 3901–4000   → 172.28.128.0/17
CDN   VLAN: 4101–4100   → 172.29.128.0/17
```

---

## 🔹 PPPoE Service Package List

| Package (Speed) | Profile Name      | Price (BDT) |
| --------------- | ----------------- | ----------- |
| 20M Regular   | Extream\_Regular  | 500         |
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

## 🔹 IP POOL & BW Shaping Que configurations Example

```
# ----------------------------
# 1) PPPoE POOLS  (/16 blocks)
# ----------------------------
/ip pool
remove [find]
add name=EXPIRED_POOL ranges=10.250.0.0/16
add name="500TK_REGULAR"     ranges=10.11.0.0/16
add name="600TK_FAMILY"      ranges=10.12.0.0/16
add name="700TK_PRIME"       ranges=10.13.0.0/16
add name="800TK_FREEDOM"     ranges=10.14.0.0/16
add name="1000TK_SILVER"     ranges=10.15.0.0/16
add name="1200TK_GOLD"       ranges=10.16.0.0/16
add name="1500TK_PLATINUM"   ranges=10.17.0.0/16
add name="2000TK_DIAMOND"    ranges=10.18.0.0/16

# ------------------------------------
# 2) ADDRESS-LISTS (match the pools)
# ------------------------------------
/ip firewall address-list
remove [find]
add list=500TK_REGULAR     address=10.11.0.0/16
add list=600TK_FAMILY      address=10.12.0.0/16
add list=700TK_PRIME       address=10.13.0.0/16
add list=800TK_FREEDOM     address=10.14.0.0/16
add list=1000TK_SILVER     address=10.15.0.0/16
add list=1200TK_GOLD       address=10.16.0.0/16
add list=1500TK_PLATINUM   address=10.17.0.0/16
add list=2000TK_DIAMOND    address=10.18.0.0/16

# ---------------------------------------------------
# 3) MANGLE: per-plan upload/download packet-marks
#     (icmp বাদ রাখা হয়েছে; passthrough=no)
# ---------------------------------------------------
/ip firewall mangle
remove [find]
# 500TK
add chain=forward src-address-list=500TK_REGULAR protocol=!icmp action=mark-packet new-packet-mark=500TK_REGULAR_UPLOAD passthrough=no
add chain=forward dst-address-list=500TK_REGULAR protocol=!icmp action=mark-packet new-packet-mark=500TK_REGULAR_DOWNLOAD passthrough=no
# 600TK
add chain=forward src-address-list=600TK_FAMILY protocol=!icmp action=mark-packet new-packet-mark=600TK_FAMILY_UPLOAD passthrough=no
add chain=forward dst-address-list=600TK_FAMILY protocol=!icmp action=mark-packet new-packet-mark=600TK_FAMILY_DOWNLOAD passthrough=no
# 700TK
add chain=forward src-address-list=700TK_PRIME protocol=!icmp action=mark-packet new-packet-mark=700TK_PRIME_UPLOAD passthrough=no
add chain=forward dst-address-list=700TK_PRIME protocol=!icmp action=mark-packet new-packet-mark=700TK_PRIME_DOWNLOAD passthrough=no
# 800TK
add chain=forward src-address-list=800TK_FREEDOM protocol=!icmp action=mark-packet new-packet-mark=800TK_FREEDOM_UPLOAD passthrough=no
add chain=forward dst-address-list=800TK_FREEDOM protocol=!icmp action=mark-packet new-packet-mark=800TK_FREEDOM_DOWNLOAD passthrough=no
# 1000TK
add chain=forward src-address-list=1000TK_SILVER protocol=!icmp action=mark-packet new-packet-mark=1000TK_SILVER_UPLOAD passthrough=no
add chain=forward dst-address-list=1000TK_SILVER protocol=!icmp action=mark-packet new-packet-mark=1000TK_SILVER_DOWNLOAD passthrough=no
# 1200TK
add chain=forward src-address-list=1200TK_GOLD protocol=!icmp action=mark-packet new-packet-mark=1200TK_GOLD_UPLOAD passthrough=no
add chain=forward dst-address-list=1200TK_GOLD protocol=!icmp action=mark-packet new-packet-mark=1200TK_GOLD_DOWNLOAD passthrough=no
# 1500TK
add chain=forward src-address-list=1500TK_PLATINUM protocol=!icmp action=mark-packet new-packet-mark=1500TK_PLATINUM_UPLOAD passthrough=no
add chain=forward dst-address-list=1500TK_PLATINUM protocol=!icmp action=mark-packet new-packet-mark=1500TK_PLATINUM_DOWNLOAD passthrough=no
# 2000TK
add chain=forward src-address-list=2000TK_DIAMOND protocol=!icmp action=mark-packet new-packet-mark=2000TK_DIAMOND_UPLOAD passthrough=no
add chain=forward dst-address-list=2000TK_DIAMOND protocol=!icmp action=mark-packet new-packet-mark=2000TK_DIAMOND_DOWNLOAD passthrough=no

# ---------------------------------------------------
# 4) QUEUE TYPES (PCQ) – names consistent everywhere
#    pcq-rate এখানে আপনার প্ল্যানের speed অনুযায়ী দিন
# ---------------------------------------------------
/queue type
# keep default ethernet-default tuning (optional)
set 1 pfifo-limit=4096

# 500TK
add name=500TK_REGULAR_DOWNLOAD kind=pcq pcq-classifier=dst-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
add name=500TK_REGULAR_UPLOAD   kind=pcq pcq-classifier=src-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
# 600TK
add name=600TK_FAMILY_DOWNLOAD  kind=pcq pcq-classifier=dst-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
add name=600TK_FAMILY_UPLOAD    kind=pcq pcq-classifier=src-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
# 700TK
add name=700TK_PRIME_DOWNLOAD   kind=pcq pcq-classifier=dst-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
add name=700TK_PRIME_UPLOAD     kind=pcq pcq-classifier=src-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
# 800TK
add name=800TK_FREEDOM_DOWNLOAD kind=pcq pcq-classifier=dst-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
add name=800TK_FREEDOM_UPLOAD   kind=pcq pcq-classifier=src-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
# 1000TK
add name=1000TK_SILVER_DOWNLOAD kind=pcq pcq-classifier=dst-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
add name=1000TK_SILVER_UPLOAD   kind=pcq pcq-classifier=src-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
# 1200TK
add name=1200TK_GOLD_DOWNLOAD   kind=pcq pcq-classifier=dst-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
add name=1200TK_GOLD_UPLOAD     kind=pcq pcq-classifier=src-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
# 1500TK
add name=1500TK_PLATINUM_DOWNLOAD kind=pcq pcq-classifier=dst-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
add name=1500TK_PLATINUM_UPLOAD   kind=pcq pcq-classifier=src-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
# 2000TK
add name=2000TK_DIAMOND_DOWNLOAD  kind=pcq pcq-classifier=dst-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB
add name=2000TK_DIAMOND_UPLOAD    kind=pcq pcq-classifier=src-address pcq-rate=0 pcq-limit=100KiB pcq-total-limit=1000000KiB

# fq-codel (যদি দরকার হয়)
/queue type
add name=wan-queue-type kind=fq-codel

# ---------------------------------------------------
# 5) INTERFACE QUEUE (optional tuning)
# ---------------------------------------------------
/queue interface
set ether1_1G queue=ethernet-default
set sfp-sfpplus1_WAN_CORE_1/3 queue=ethernet-default
set sfp-sfpplus2_WAN_CORE_1/4 queue=ethernet-default

# ---------------------------------------------------
# 6) QUEUE TREE — names fixed, queues exist
#     Upload parent: আপনার WAN-UP interface
#     Download parent: global (বা LAN-bridge)
# ---------------------------------------------------
/queue tree
remove [find]

# 500TK
add name=500TK_REGULAR_UPLOAD   parent=sfp-sfpplus1_WAN_CORE_1/3 packet-mark=500TK_REGULAR_UPLOAD   queue=500TK_REGULAR_UPLOAD   priority=4 bucket-size=0.15
add name=500TK_REGULAR_DOWNLOAD parent=global                         packet-mark=500TK_REGULAR_DOWNLOAD queue=500TK_REGULAR_DOWNLOAD priority=4 bucket-size=0.15

# 600TK
add name=600TK_FAMILY_UPLOAD    parent=sfp-sfpplus1_WAN_CORE_1/3 packet-mark=600TK_FAMILY_UPLOAD    queue=600TK_FAMILY_UPLOAD    priority=4 bucket-size=0.15
add name=600TK_FAMILY_DOWNLOAD  parent=global                         packet-mark=600TK_FAMILY_DOWNLOAD  queue=600TK_FAMILY_DOWNLOAD  priority=4 bucket-size=0.15

# 700TK
add name=700TK_PRIME_UPLOAD     parent=sfp-sfpplus1_WAN_CORE_1/3 packet-mark=700TK_PRIME_UPLOAD     queue=700TK_PRIME_UPLOAD     priority=3 bucket-size=0.20
add name=700TK_PRIME_DOWNLOAD   parent=global                         packet-mark=700TK_PRIME_DOWNLOAD   queue=700TK_PRIME_DOWNLOAD   priority=3 bucket-size=0.20

# 800TK
add name=800TK_FREEDOM_UPLOAD   parent=sfp-sfpplus1_WAN_CORE_1/3 packet-mark=800TK_FREEDOM_UPLOAD   queue=800TK_FREEDOM_UPLOAD   priority=3 bucket-size=0.20
add name=800TK_FREEDOM_DOWNLOAD parent=global                         packet-mark=800TK_FREEDOM_DOWNLOAD queue=800TK_FREEDOM_DOWNLOAD priority=3 bucket-size=0.20

# 1000TK
add name=1000TK_SILVER_UPLOAD   parent=sfp-sfpplus1_WAN_CORE_1/3 packet-mark=1000TK_SILVER_UPLOAD   queue=1000TK_SILVER_UPLOAD   priority=2 bucket-size=0.22
add name=1000TK_SILVER_DOWNLOAD parent=global                         packet-mark=1000TK_SILVER_DOWNLOAD queue=1000TK_SILVER_DOWNLOAD priority=2 bucket-size=0.22

# 1200TK
add name=1200TK_GOLD_UPLOAD     parent=sfp-sfpplus1_WAN_CORE_1/3 packet-mark=1200TK_GOLD_UPLOAD     queue=1200TK_GOLD_UPLOAD     priority=2 bucket-size=0.22
add name=1200TK_GOLD_DOWNLOAD   parent=global                         packet-mark=1200TK_GOLD_DOWNLOAD   queue=1200TK_GOLD_DOWNLOAD   priority=2 bucket-size=0.22

# 1500TK
add name=1500TK_PLATINUM_UPLOAD   parent=sfp-sfpplus1_WAN_CORE_1/3 packet-mark=1500TK_PLATINUM_UPLOAD   queue=1500TK_PLATINUM_UPLOAD   priority=1 bucket-size=0.22
add name=1500TK_PLATINUM_DOWNLOAD parent=global                         packet-mark=1500TK_PLATINUM_DOWNLOAD queue=1500TK_PLATINUM_DOWNLOAD priority=1 bucket-size=0.22

# 2000TK
add name=2000TK_DIAMOND_UPLOAD    parent=sfp-sfpplus1_WAN_CORE_1/3 packet-mark=2000TK_DIAMOND_UPLOAD    queue=2000TK_DIAMOND_UPLOAD    priority=1 bucket-size=0.25
add name=2000TK_DIAMOND_DOWNLOAD  parent=global                         packet-mark=2000TK_DIAMOND_DOWNLOAD  queue=2000TK_DIAMOND_DOWNLOAD  priority=1 bucket-size=0.25

```
