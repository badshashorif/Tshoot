IIG RTR এবং BDIX VRF‑এর মধ্যে **VRF‑ভিত্তিক iBGP peering** কনফিগারেশনের জন্য আমরা **In/Out**—দুই পাশেই মোট **৪টি route‑policy** ব্যবহার করব। ভবিষ্যতে **BGP attribute tuning, prefix diversion/transition** এবং **security** সহজে নিশ্চিত করার জন্য নেমিং ও কাঠামো নিচে স্ট্যান্ডার্ড করা হলো।

***

## ✅ Policy Naming Convention (৪টি policy)

*   **IIG রাউটার থেকে (peering: BDIX VRF)**
    *   **In:** `BDIX_VRF_IN`
    *   **Out:** `BDIX_VRF_OUT`

*   **BDIX VRF থেকে (peering: IIG RTR)**
    *   **In:** `IIG_BDIX_VRF_IN`
    *   **Out:** `IIG_BDIX_VRF_OUT`

> **Prefix‑lists (উদাহরণ):**
>
> *   `DEMO_PREFIX_LIST` → টেস্ট/ডেমো: `10.0.0.0/32`
> *   `RETAIL_IPV4_PREFIX_LIST` → প্রোডাকশন রিটেইল ব্লক: `122.99.100.0/22` + চাইল্ড /24‑গুলো

***

## 🎯 উদ্দেশ্য (শর্ট)

*   **In‑policy:** পার্টনার/পিয়ার থেকে **যে প্রিফিক্সগুলো ইনবাউন্ড অনুমোদন** পাবো তা ফিল্টার/ট্যাগ করা।
*   **Out‑policy:** আমাদের পক্ষ থেকে **যে প্রিফিক্সগুলো এক্সপোর্ট** করবো তা নিয়ন্ত্রণ করা।
*   ভবিষ্যতে প্রয়োজন হলে **local‑pref/MED/community** সেট করে **ডাইভারশন/ট্রানজিশন** করা যাবে।
*   **সিকিউরিটি:** অবাঞ্ছিত প্রিফিক্স/লিক এড়াতে ডিফল্ট **ড্রপ**।

***

## 🔧 BGP Configuration — **FROM IIG\_RTR\_END**

```bash
#show running-config | begin neighbor 172.30.250.14

neighbor 172.30.250.14
  remote-as 17471
  description BDIX_VRF_P2P
  update-source TenGigE0/0/0/12.75
  address-family ipv4 unicast
   route-policy BDIX_VRF_IN in
   route-policy BDIX_VRF_OUT out
   soft-reconfiguration inbound always
  !
```

**Route‑policy (IN):**

```bash
#show running-config route-policy BDIX_VRF_IN

route-policy BDIX_VRF_IN
  if destination in DEMO_PREFIX_LIST then
    pass
  else
    drop
  endif
end-policy
!
```

**Prefix‑set (DEMO):**

```bash
#show running-config prefix-set DEMO_PREFIX_LIST
prefix-set DEMO_PREFIX_LIST
  10.0.0.0/32
end-set
!
```

**Route‑policy (OUT):**

```bash
#show running-config route-policy BDIX_VRF_OUT
route-policy BDIX_VRF_OUT
  if destination in RETAIL_IPV4_PREFIX_LIST then
    pass
  else
    drop
  endif
end-policy
!
```

**Prefix‑set (RETAIL):**

```bash
#show running-config prefix-set RETAIL_IPV4_PREFIX_LIST
prefix-set RETAIL_IPV4_PREFIX_LIST
  122.99.100.0/22,
  122.99.100.0/24,
  122.99.101.0/24,
  122.99.102.0/24,
  122.99.103.0/24
end-set
!
```

***

## 🔧 BGP Configuration — **FROM BDIX\_VRF\_END**

```bash
#show running-config | begin neighbor 172.30.250.13
vrf BDIX
 neighbor 172.30.250.13
  remote-as 17471
  description BDIX_VRF_P2P
  update-source TenGigE0/0/0/12.75
  address-family ipv4 unicast
   route-policy IIG_BDIX_VRF_IN in
   route-policy IIG_BDIX_VRF_OUT out
   soft-reconfiguration inbound always
  !
```

**Route‑policy (IN):**

```bash
#show running-config route-policy IIG_BDIX_VRF_IN

route-policy IIG_BDIX_VRF_IN
  if destination in RETAIL_IPV4_PREFIX_LIST then
    pass
  else
    drop
  endif
end-policy
!
```

**Prefix‑set (RETAIL):**

```bash
#show running-config prefix-set RETAIL_IPV4_PREFIX_LIST
prefix-set RETAIL_IPV4_PREFIX_LIST
  122.99.100.0/22,
  122.99.100.0/24,
  122.99.101.0/24,
  122.99.102.0/24,
  122.99.103.0/24
end-set
!
```

**Route‑policy (OUT):**

```bash
#show running-config route-policy IIG_BDIX_VRF_OUT
route-policy IIG_BDIX_VRF_OUT
  if destination in DEMO_PREFIX_LIST then
    pass
  else
    drop
  endif
end-policy
!
```

**Prefix‑set (DEMO):**

```bash
#show running-config prefix-set DEMO_PREFIX_LIST
prefix-set DEMO_PREFIX_LIST
  10.0.0.0/32
end-set
!
```

***

## 📌 ভবিষ্যতের জন্য জায়গা রাখা (attribute/diversion/transition)

প্রয়োজনে যে কোনো policy‑র ভেতরে নিচের লাইনগুলো যোগ করে **ট্রাফিক‑ডাইভারশন/ট্রানজিশন/সিকিউরিটি** করা যাবে—এখন কমেন্টেড উদাহরণ:

```rpl
route-policy BDIX_VRF_IN
  if destination in DEMO_PREFIX_LIST then
    # set local-preference 200              # ইনবাউন্ড ট্রাফিক প্রেফারেন্স
    # set med 50                            # রুট সিলেকশন টিউনিং
    # set community (65000:100) additive    # ট্যাগ/ডাইভারশন
    pass
  else
    drop
  endif
end-policy
```

***

## 🔐 অপারেশনাল নোটস (সিকিউরিটি/সেফটি)

*   **BGP password** (MD5) ব্যবহার করুন: `neighbor x.x.x.x password <secret>`
*   **TTL‑security (GTSM)** বিবেচনা করুন: `neighbor x.x.x.x ttl-security hops 1`
*   **Max‑prefix** গার্ড: `address-family ipv4 unicast; maximum-prefix <N> warning-only`
*   Route‑refresh সমর্থন থাকলে `soft-reconfiguration inbound always` এড়িয়ে **ডায়নামিক রিফ্রেশ** প্রেফার করুন।
*   Policy/Prefix‑list পরিবর্তন হলে **চেঞ্জ‑লগ + মেইনটেন্যান্স উইন্ডো** অনুসরণ করুন।


