<div align="center">

# 🔌 802.1Q Trunking and Native VLAN Mismatch

### Switching Series · Lab 1 · Cisco Packet Tracer

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-049fd9?style=for-the-badge)
![Trunk](https://img.shields.io/badge/Trunk-802.1Q-red?style=for-the-badge)
![Native VLAN](https://img.shields.io/badge/Native%20VLAN-99-orange?style=for-the-badge)
![CDP](https://img.shields.io/badge/CDP-Mismatch%20Detected-yellow?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Verified-success?style=for-the-badge)

*One cable. Many VLANs. And both ends must agree on the untagged one.*

</div>

---

## 📑 Table of Contents
1. [Overview](#-overview)
2. [Topology](#-topology)
3. [Port Summary](#-port-summary)
4. [Trunking Theory](#-trunking-theory)
5. [Configuration](#-configuration)
6. [Verification: Stage by Stage](#-verification-stage-by-stage)
7. [Native VLAN Mismatch Explained](#-native-vlan-mismatch-explained)
8. [Before vs After](#-before-vs-after)
9. [Observations](#-observations)
10. [Key Takeaways](#-key-takeaways)
11. [Troubleshooting Notes](#-troubleshooting-notes)
12. [Next Labs](#-next-labs-in-the-switching-series)
13. [Author](#-author)

---

## 🎯 Overview

A **trunk** link carries traffic for **many VLANs** over one cable between switches. **802.1Q (dot1q)** is the standard that tags frames with a VLAN ID so the receiving switch knows which VLAN each frame belongs to.

This lab builds a dot1q trunk between two **Cisco 3560-24PS** switches and then shows what happens when the **native VLAN does not match** on both ends.

| Item | Detail |
|---|---|
| Switches | SW-1 (labelled **A**) and SW-2 (labelled **B**), both 3560-24PS |
| Trunk link | GigabitEthernet0/2 on both switches |
| Encapsulation | **802.1Q (dot1q)** |
| Allowed VLANs | 1, 10, 20, 30 |
| Native VLAN | **99** |
| DTP | Disabled with `nonegotiate` |
| Issue demonstrated | `%CDP-4-NATIVE_VLAN_MISMATCH` |

---

## 🗺 Topology

```
        SW-1 (A)                                       SW-2 (B)
      ┌──────────┐   gig0/2                 gig0/2   ┌──────────┐
      │ 3560-24PS│━━━━━━━━ 802.1Q trunk (dot1q) ━━━━━━│ 3560-24PS│
      └──────────┘   native VLAN 99     native VLAN 99└──────────┘
                     allowed VLANs: 1, 10, 20, 30
```

---

## 📋 Port Summary

| Device | Port | Role | Encapsulation | Native VLAN | Allowed VLANs |
|---|---|---|---|---|---|
| SW-1 (A) | Gig0/2 | Trunk | dot1q | 99 | 1, 10, 20, 30 |
| SW-2 (B) | Gig0/2 | Trunk | dot1q | 99 | 1, 10, 20, 30 |

---

## 📚 Trunking Theory

### How 802.1Q works

| Concept | Meaning |
|---|---|
| Access port | Carries **one** VLAN, with untagged frames |
| Trunk port | Carries **many** VLANs on one link |
| **802.1Q tag** | A 4-byte tag inserted in the Ethernet frame, including a 12-bit **VLAN ID** |
| **Native VLAN** | The one VLAN whose frames cross the trunk **untagged** (default: VLAN 1) |
| Allowed VLAN list | Which VLANs may use the trunk (default: all) |

```
 Tagged frame (VLAN 10):   [ Dest | Src | 802.1Q tag: VLAN 10 | Type | Data | FCS ]
 Native VLAN frame (99):   [ Dest | Src |        (no tag)       | Type | Data | FCS ]
```

### Why `switchport trunk encapsulation dot1q` is needed here

| Switch family | Encapsulations supported | Command needed? |
|---|---|---|
| **3560 (this lab)** | ISL **and** 802.1Q | ✅ Yes: you must choose before `mode trunk` |
| 2960 | 802.1Q only | ❌ No: the command does not exist |

### DTP (Dynamic Trunking Protocol)

| Mode | Behaviour |
|---|---|
| `dynamic auto` (**default**) | Becomes a trunk only if the other side asks |
| `dynamic desirable` | Actively asks the other side to trunk |
| `trunk` | Always a trunk |
| `access` | Never a trunk |
| `nonegotiate` | Stops sending DTP frames (use with `trunk` or `access`) |

> Two `dynamic auto` ports will **never** form a trunk, because both wait for the other to ask.

---

## ⚙️ Configuration

### SW-1 (configured first)

```
SW-1(config)# interface gigabitEthernet 0/2
SW-1(config-if)# switchport trunk encapsulation dot1q
SW-1(config-if)# switchport mode trunk
SW-1(config-if)# switchport trunk allowed vlan 1,10,20,30
SW-1(config-if)# switchport trunk native vlan 99
SW-1(config-if)# switchport nonegotiate
SW-1(config-if)# end
```

### SW-2

```
SW-2(config)# interface gigabitEthernet 0/2
SW-2(config-if)# switchport trunk encapsulation dot1q
SW-2(config-if)# switchport mode trunk
SW-2(config-if)# switchport trunk allowed vlan 1,10,20,30
%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on
GigabitEthernet0/2 (1), with SW-1 GigabitEthernet0/2 (99)      ← the warning
SW-2(config-if)# switchport trunk native vlan 99               ← the fix
SW-2(config-if)# switchport nonegotiate
SW-2(config-if)# end
```

### Config breakdown

| Command | Purpose |
|---|---|
| `switchport trunk encapsulation dot1q` | Choose 802.1Q (required on a 3560 before trunk mode) |
| `switchport mode trunk` | Force the port into trunk mode |
| `switchport trunk allowed vlan 1,10,20,30` | Only these VLANs may cross the trunk |
| `switchport trunk native vlan 99` | VLAN 99 is the untagged VLAN on this trunk |
| `switchport nonegotiate` | Disable DTP negotiation on the port |

---

## ✅ Verification: Stage by Stage

### 🟡 Stage 1: Default state before any trunk configuration (SW-1)

```
SW-1# show interfaces gigabitEthernet 0/2 switchport
Name: Gig0/2
Switchport: Enabled
Administrative Mode: dynamic auto
Operational Mode: static access
Administrative Trunking Encapsulation: negotiated
Operational Trunking Encapsulation: native
Negotiation of Trunking: On
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 1 (default)
Trunking VLANs Enabled: All
```

**Annotations**
- **Administrative Mode: dynamic auto**: the port waits for the other side to ask for a trunk.
- **Operational Mode: static access**: right now it is **not trunking**.
- **Administrative Trunking Encapsulation: negotiated**: no encapsulation has been chosen yet.
- **Negotiation of Trunking: On**: DTP is active.
- **Native VLAN 1** and **all VLANs enabled**: these are the factory defaults we are about to change.

---

### 🔧 Stage 2: Configure SW-1

```
SW-1(config-if)# switchport trunk encapsulation dot1q
SW-1(config-if)# switchport mode trunk
SW-1(config-if)# switchport trunk allowed vlan 1,10,20,30
SW-1(config-if)# switchport trunk native vlan 99
SW-1(config-if)# switchport nonegotiate
```

**Annotation:** SW-1 now uses native VLAN **99**, but SW-2 has not been configured yet, which is where the mismatch comes from.

---

### 🔴 Stage 3: Configure SW-2 → CDP warns about the mismatch

```
SW-2(config-if)# switchport trunk encapsulation dot1q
SW-2(config-if)# switchport mode trunk
SW-2(config-if)# switchport trunk allowed vlan 1,10,20,30

%CDP-4-NATIVE_VLAN_MISMATCH: Native VLAN mismatch discovered on
GigabitEthernet0/2 (1), with SW-1 GigabitEthernet0/2 (99)
```

**Annotations**
- The message appears on **SW-2**.
- **`(1)`** is SW-2's own native VLAN, still the default.
- **`(99)`** is the native VLAN CDP learned from **SW-1**.
- The two switches **disagree** about which VLAN is untagged on this link.
- In your own words from the lab: *"I configured native VLAN 99 on SW-1. When I configure the native VLAN on SW-2, this error will be solved."*

---

### 🟢 Stage 4: Fix the mismatch on SW-2

```
SW-2(config-if)# switchport trunk native vlan 99
SW-2(config-if)# switchport nonegotiate
```

**Annotation:** With **native VLAN 99 on both ends**, no further CDP warning appears: **no error, no mismatch**.

---

### 🔎 Stage 5: Final state on SW-1

```
SW-1# show interfaces gigabitEthernet 0/2 switchport
Name: Gig0/2
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: Off
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 99 (Inactive)
Trunking VLANs Enabled: 1,10,20,30
```

---

### 🔎 Stage 6: Final state on SW-2

```
SW-2# show interfaces gigabitEthernet 0/2 switchport
Name: Gig0/2
Switchport: Enabled
Administrative Mode: trunk
Operational Mode: trunk
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: dot1q
Negotiation of Trunking: Off
Access Mode VLAN: 1 (default)
Trunking Native Mode VLAN: 99 (Inactive)
Trunking VLANs Enabled: 1,10,20,30
```

**Annotation:** SW-2 now mirrors SW-1 exactly. Both ends agree on mode, encapsulation, native VLAN and allowed VLANs.

---

### 📊 Stage 7: The trunk summary (SW-1)

```
SW-1# show interfaces trunk

Port      Mode   Encapsulation  Status     Native vlan
Gig0/2    on     802.1q         trunking   99

Port      Vlans allowed on trunk
Gig0/2    1,10,20,30

Port      Vlans allowed and active in management domain
Gig0/2    1

Port      Vlans in spanning tree forwarding state and not pruned
Gig0/2    1
```

| Section | Value | Meaning |
|---|---|---|
| Mode | **on** | Hard-configured trunk (no DTP) |
| Encapsulation | **802.1q** | The tagging standard chosen |
| Status | **trunking** | The trunk is up ✅ |
| Native vlan | **99** | Matches the other switch ✅ |
| Vlans allowed on trunk | 1,10,20,30 | The allowed list we set |
| Allowed **and active** | **1** | Only VLAN 1 exists so far. See [Observations](#-observations) |
| Spanning tree forwarding | **1** | Only VLAN 1 is forwarding as a result |

---

## 🧨 Native VLAN Mismatch Explained

```
   SW-1                                         SW-2
   native VLAN 99 ━━━━━ untagged frames ━━━━━ native VLAN 1
        │                                         │
   sends untagged frame ──────────────────►  treats it as VLAN 1  ✖
```

| Question | Answer |
|---|---|
| What is it? | The two ends of a trunk use **different native VLANs** |
| Why is it a problem? | Untagged frames from one VLAN are placed into a **different VLAN** on the other side |
| Possible effects | Traffic leaking between VLANs, connectivity problems, **VLAN-hopping risk** |
| How is it detected? | **CDP** logs `%CDP-4-NATIVE_VLAN_MISMATCH` and names both values |
| How is it fixed? | Set the **same** native VLAN on **both** ends |
| Best practice | Use a **dedicated, unused VLAN** (such as 99) as the native VLAN |

---

## ⚖️ Before vs After

| Setting | Before (default) | After (configured) |
|---|---|---|
| Administrative Mode | dynamic auto | **trunk** |
| Operational Mode | static access | **trunk** |
| Administrative encapsulation | negotiated | **dot1q** |
| Operational encapsulation | native | **dot1q** |
| Negotiation of trunking | On | **Off** |
| Native VLAN | 1 (default) | **99** |
| Trunking VLANs Enabled | All | **1,10,20,30** |
| Native VLAN agreement | n/a | ✅ Same on both ends |

---

## 🔍 Observations

| Observation | What it means | Next step |
|---|---|---|
| Native VLAN shows **99 (Inactive)** | VLAN 99 **does not exist yet** in the VLAN database | Create VLAN 99 on both switches |
| "Allowed and active" shows only **1** | VLANs 10, 20 and 30 are allowed on the trunk but **not created** | Create them, and they will start to forward |
| Spanning tree forwarding: **1** | Only VLAN 1 is carrying traffic today | Same as above |

Suggested follow-up (not part of this lab's screenshots):

```
SW-1(config)# vlan 10
SW-1(config-vlan)# name SALES
SW-1(config)# vlan 20
SW-1(config-vlan)# name HR
SW-1(config)# vlan 30
SW-1(config-vlan)# name IT
SW-1(config)# vlan 99
SW-1(config-vlan)# name NATIVE
(repeat on SW-2)
```

---

## 💡 Key Takeaways

| # | Takeaway |
|---|---|
| 1 | A trunk carries many VLANs over one link, using **802.1Q tags** |
| 2 | A **3560** must have `switchport trunk encapsulation dot1q` set before `mode trunk` |
| 3 | Default **dynamic auto** does not form a trunk on its own |
| 4 | `switchport nonegotiate` turns **DTP** off |
| 5 | The **native VLAN** carries untagged frames and must **match** on both ends |
| 6 | **CDP** reports the mismatch and shows **both** native VLAN values |
| 7 | `show interfaces switchport` and `show interfaces trunk` prove the final state |
| 8 | A VLAN must **exist** before it becomes active on a trunk |

---

## 🛠 Troubleshooting Notes

| Symptom | Check |
|---|---|
| Port never becomes a trunk | Is it `dynamic auto` on both sides? Set `mode trunk` on at least one side |
| `switchport mode trunk` rejected | On a 3560, set `switchport trunk encapsulation dot1q` first |
| `%CDP-4-NATIVE_VLAN_MISMATCH` | Native VLAN differs: set the same value on both ends |
| `nonegotiate` rejected | The port must be `access` or `trunk` first |
| A VLAN's traffic doesn't cross | Is it in the allowed list **and** created on both switches? |
| Native VLAN shows `Inactive` | The VLAN is not created yet |
| One side tags, the other doesn't | Compare encapsulation and native VLAN on both ends |

---

## 🚀 Next Labs in the Switching Series

- [x] Lab 1: 802.1Q trunk and native VLAN mismatch ✅
- [ ] Create VLANs 10, 20, 30 and 99, and assign access ports
- [ ] Test hosts in the same VLAN across the trunk
- [ ] DTP modes: auto, desirable and nonegotiate compared
- [ ] Inter-VLAN routing (router-on-a-stick)
- [ ] VTP and spanning tree basics

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-certified Network & NOC Engineer · Lower Dir, KPK, Pakistan

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github)](https://github.com/maazkhanms)

⭐ If this lab helped you, give the repo a star!
