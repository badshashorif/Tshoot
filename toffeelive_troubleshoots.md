##আমাদের NCS কোর রাউটার থেকে /22–/24 প্রিফিক্স অ্যানাউন্স করা আছে। নির্দিষ্টভাবে https://toffeelive.com
 সাইটটি স্বাভাবিকভাবে লোড হয়, পিং ও ট্রেসরুটও সফল; কিন্তু কোনো ভিডিও/স্ট্রিমিং কনটেন্ট প্লে হয় না। সমস্যা কেবল আমাদের প্রিফিক্স/সোর্স IP থেকে দেখা যাচ্ছে; অন্য নেটওয়ার্কে প্লেব্যাক ঠিকমতো কাজ করে।


#বোঝা যাচ্ছে—ওয়েবসাইট (toffeelive.com) লোড হচ্ছে, পিং/ট্রেসও ঠিক; কিন্তু শুধু ভিডিও/স্ট্রিম প্লে হচ্ছে না এবং এটা আপনার নিজের announced prefix (/22–/24) থেকে ঘটছে। সাধারণত এ রকম ক্ষেত্রে রুট কজ পড়ে ৪টা জায়গায়:

1. **IP reputation/geo বা ASN-based block** (CDN/OTT আপনার পাবলিক IP-কে “datacenter/VPN” হিসেবে ধরছে বা দেশের বাইরে দেখছে)
2. **QUIC (UDP/443) বা DRM/license ডোমেইন ব্লক** (ফায়ারওয়াল/NAT-এ UDP443 বা কিছু সাবডোমেইন অনিচ্ছাকৃতভাবে ব্লক)
3. **MSS/PMTU/ICMP ব্লক** (PPPoE ইত্যাদিতে MTU mismatch, ICMP “fragmentation needed” ড্রপ হলে ভিডিও চাংক stall করে)
4. **NAT stickiness/ECMP issue** (একই সেশনের segment রিকোয়েস্ট বিভিন্ন public IP-তে বেরোচ্ছে—CDN টোকেন/সেশন invalid ধরে 403 দেয়)

নিচে দ্রুত **ডায়াগনসিস → ফিক্স** রানবুক দিলাম। একবারেই করে ফেলুন—আলাদা অপেক্ষার দরকার নেই।

---

## 0) দ্রুত স্কোপ চেক

* **একই PC/মোবাইল থেকে অন্য ISP (মোবাইল ডাটা)**: ভিডিও প্লে হয় কিনা দেখুন। হলে আপনার প্রিফিক্স-সাইড ইস্যু।
* **Prefix-ভিত্তিক**: আপনার নেটওয়ার্কে অন্য public IP (ভিন্ন /32) থেকে টেস্ট করুন—কিছু IP কাজ করে, কিছু করে না—এই pattern থাকলে **IP reputation/stickiness** সবচেয়ে সম্ভাব্য।

---

## 1) ব্রাউজার DevTools (Network tab) দিয়ে Error ধরুন (সবচেয়ে জরুরি)

* ভিডিও চালু করুন → Network ট্যাবে **.m3u8, .mpd, .ts, .m4s, key/license** রিকোয়েস্টে কী স্ট্যাটাস কোড?

  * **403/451**: প্রায় নিশ্চিত **geo/reputation/ASN block** বা **token mismatch (stickiness)**।
  * **0 / (failed)** বা **timeout**: **UDP/443 (QUIC) বা DRM সাবডোমেইন** ব্লক/ড্রপ।
  * **200 কিন্তু কিছুক্ষণ পর stall**: **MSS/PMTU/ICMP** সমস্যা সম্ভাব্য।

> DevTools থেকে **Copy > Copy as cURL** নিন—নিচের CLI টেস্টে কাজে লাগবে।

---

## 2) QUIC / UDP: 443 খোলা তো?

কিছু OTT/CDN ডিফল্টে HTTP/3 (QUIC) দেয়, fallback সবসময় ঠিকমতো না-ও হতে পারে।

* টেস্ট (Linux):

  ```bash
  # UDP 443 traceroute (path দেখুন)
  traceroute -U -p 443 <cdn-hostname>

  # HTTP/3 সমর্থন test (fallback বুঝতে)
  curl -I --http3 https://<cdn-hostname>/
  curl -I --http2 https://<cdn-hostname>/
  ```

* **Firewall/NAT এ নিশ্চিত করুন**:

  * **UDP/443** allow (forward+nat chain)
  * কোন DPI/IDS QUIC ব্লক করছে কিনা দেখুন

**MikroTik example**:

```rsc
/ip firewall filter add chain=forward protocol=udp dst-port=443 action=accept comment="Allow QUIC"
/ip firewall filter add chain=input protocol=icmp action=accept comment="Allow ICMP incl. frag-needed"
```

---

## 3) DRM/License ও সাবডোমেইন allowlist

অনেক সময় প্লেয়ার **main domain**-এ না গিয়ে **drm/license/sub-cdn** ডোমেইনে হিট করে। সেগুলো ব্লক হলে প্লে ব্যর্থ।

* DevTools → failing **hostnames** কপি করুন।
* সেই ডোমেইনগুলো DNS/Firewall-এ ব্লক হচ্ছে কিনা দেখুন। Proxy/Caching থাকলে **bypass** করুন।

**টেস্ট**:

```bash
nslookup <failing-host>
curl -v https://<failing-host>/ --max-time 10
```

---

## 4) NAT stickiness / বহু public IP সমস্যা

HLS/DASH অনেক ছোট ছোট HTTP রিকোয়েস্ট করে। যদি আপনার CGNAT/ECMP/NTH/load-balance-এর কারণে **একই ইউজারের বিভিন্ন রিকোয়েস্ট ভিন্ন public IP** দিয়ে বের হয়, CDN এটাকে “account sharing/VPN” ধরে **403** দিতেই পারে।

**সমাধান**:

* ঐ সাইট/সাবডোমেইনগুলোর জন্য **policy routing** করে **একই egress IP** use করুন (sticky NAT)।
* Per-user বা per-/24 basis **static src-nat** দিন।

**MikroTik (example, নির্দিষ্ট CDN host list ধরে)**:

```rsc
/ip firewall address-list
add list=TOFFEE-CDN address=<cdn1>
add list=TOFFEE-CDN address=<cdn2>
# ... (DevTools থেকে যা পাবেন)

/ip firewall mangle
add chain=prerouting dst-address-list=TOFFEE-CDN action=mark-connection new-connection-mark=toffee_conn
add chain=prerouting connection-mark=toffee_conn action=mark-routing new-routing-mark=toffee_route

/ip route
add dst-address=0.0.0.0/0 gateway=<wan-gw-for-IP-A> routing-mark=toffee_route

/ip firewall nat
add chain=srcnat dst-address-list=TOFFEE-CDN action=src-nat to-addresses=<Public-IP-A> comment="Stick to one IP"
```

---

## 5) MSS/PMTU/ICMP (বিশেষ করে PPPoE হলে)

PPPoE-তে effective MTU সাধারণত **1492**; ICMP “frag needed” ব্লক হলে বড় TLS রেকর্ড/ভিডিও চাংক আটকে যায়।

**টেস্ট**:

```bash
# PMTU probe (1472=1500-28) DF সেট করে
ping -M do -s 1472 <cdn-ip>
# কমাতে কমাতে দেখুন কোন সাইজে যায়
```

**ফিক্স**:

* **ICMP allow** করুন (input+forward) বিশেষ করে type 3 code 4 (“frag needed”)।
* **TCP MSS clamp** দিন।

**MikroTik**:

```rsc
/ip firewall mangle
add chain=forward protocol=tcp tcp-flags=syn action=change-mss new-mss=clamp-to-pmtu comment="Clamp MSS"
```

**Cisco (IOS/IOS-XE; XR-এ interface-এর নিচে সমতুল্য কমান্ড খুঁজে নিন)**:

```cisco
interface <pppoe/lan>
 ip tcp adjust-mss 1452
```

---

## 6) IP reputation / Geo (সবচেয়ে কমন রুট কজ)

অনেক OTT/CDN **ডেটা সেন্টার/প্রক্সি IP** ব্লক করে। আপনার egress IP-গুলো যদি MaxMind/DB-IP/IP2Location-এ “hosting/VPN” বা ভুল দেশে দেখায়, প্লে বন্ধ হবে।

**যা করবেন**:

* **Check** (আপনার গেটওয়ে থেকে চালান):

  ```bash
  curl https://ifconfig.co/json
  curl https://ipinfo.io/json
  ```

  দেখুন **country=BD**, org=**AS17471 Grameen Cybernet** ইত্যাদি ঠিক আছে কিনা।

* **Update/Correct**:

  * MaxMind GeoIP correction form
  * DB-IP / IP2Location correction
  * ipinfo / ipregistry support-এ টিকিট
  * **rDNS (PTR)** দিন “broadband/dynamic/isp” টাইপ নামকরণে (ভুলে “server/vps” টাইপ রাখবেন না)
  * **IRR route** object ও **RPKI ROA** সঠিক আছে কিনা দেখুন
  * WHOIS-এ আপনার **org/descr** ISP হিসেবে পরিষ্কার করুন

* **Toffee/CDN-এ টিকিট**: আপনার প্রিফিক্স(/22–/24), egress /32 লিস্ট, ASN **AS17471**, উদাহরণ **timestamp, public IP, HTTP status (403/0)**, এবং DevTools-এর **HAR** অ্যাটাচ করুন। রিক্ল্যাসিফাই করতে বলুন।

**ইমেল টেমপ্লেট (সংক্ষিপ্ত):**

```
Subject: Playback blocked from AS17471 (Grameen Cybernet) – IP reclassification request

Hello Toffee/CDN team,
From our ISP prefix <x.x.x.0/22, y.y.y.0/24> (AS17471), Toffee webpage loads but video segments fail (HTTP 403 / request fails). From other ISPs, it plays fine.

Please verify and reclassify our egress IPs as ISP residential/business, not hosting/VPN. 
Sample IPs: A.B.C.D, E.F.G.H
Timestamps: 2025-08-19 10:30–11:00 BST
Failing hosts: <list from DevTools>

We can provide HAR/pcap if needed.

Thanks,
Grameen Cybernet (AS17471)
NOC: noc@...
```

---

## 7) Transparent proxy/caching bypass

যদি আপনার নেটে কোনো proxy/caching (Squid/NGINX, DPI, UTM) থাকে, **toffeelive.com ও তার CDN/drm সাবডোমেইন** bypass করুন। বিশেষ করে **Range headers** বা **TLS SNI**-তে হাত পড়লে স্ট্রিম ভেঙে যায়।

---

## 8) রাউটার/ইন্টারফেস কিউ/ড্রপ

ভারি পিক টাইমে shaper/queue oversubscription বা কোনো **ACL/policy-map counter** হিট করছে কিনা দেখুন।

**Cisco (XR/IOS-XE)**:

```cisco
show interface <wan> | i drops|errors
show policy-map interface <wan>
show access-lists | i <cdn-ip>
```

---

# Quick Action Checklist (৫ মিনিটে কী করবেন)

1. DevTools → failing request-এর **status code ও hostname** ধরুন।
2. **UDP/443 allow** আছে কিনা নিশ্চিত করুন; `curl --http3` / traceroute -U দিয়ে টেস্ট।
3. **MSS clamp + ICMP allow** দিন।
4. Toffee/CDN **host list** নিয়ে **sticky NAT** (এক egress IP) enforce করুন।
5. আপনার egress IP-গুলোর **geo/reputation** যাচাই করুন; ভুল হলে correction + Toffee/CDN-এ টিকিট।
6. কোনো **proxy/DPI** থাকলে hostnames **bypass** করুন।
