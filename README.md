Here lies info on enabling features on your new (and very old) Mellanox SX6036 switch.

# Shell Access
To get shell access you need to install a license to enable restricted commands. License install can also be done through the GUI. To SSH to the switch you may need to enable legacy crypto algorithms if using modern Linux. The default login is admin:admin

To install from SSH, use these commands and verify the output:
```
mellanox-sx6036 [standalone: master] > enable
mellanox-sx6036 [standalone: master] # configure terminal 
mellanox-sx6036 [standalone: master] (config) # license install LK2-RESTRICTED_CMDS_GEN2-88A1-NEWD-BPNB-1
mellanox-sx6036 [standalone: master] (config) # exit
mellanox-sx6036 [standalone: master] # show licenses
License 1: LK2-RESTRICTED_CMDS_GEN2-88A1-NEWD-BPNB-1
   Feature:          RESTRICTED_CMDS_GEN2
   Description:      Access to restricted system functionality
   Valid:            yes
   Active:           yes
```

Then to access bash:
```
mellanox-sx6036 [standalone: master] > enable
mellanox-sx6036 [standalone: master] # _shell
[admin@mellanox-sx6036 ~]#
```

Then from there you can access the genlicense tool:
```
[admin@mellanox-sx6036 ~]# cd /opt/tms/bin
[admin@mellanox-sx6036 bin]# ./genlicense 2 EFM_SX <secret> -o <option ID> <option value>
```

## Here's what each informational option ID controls:

| ID | Name | Meaning |
|----|------|---------|
| **48** | `efm_sx_max_num_hca_ports` | Maximum number of Host Channel Adapter ports the license allows. Relevant for gateway/bridging modes where the switch connects to HCAs. |
| **50** | `efm_sx_active_ports` | Total number of switch ports the license enables. Used for port-count-limited licenses (e.g., license only enables 18 of 36 ports). |
| **51** | `efm_sx_full_l2_enabled` | Enables full Layer 2 Ethernet features—MAC learning, VLANs, spanning tree, LAG, etc. Boolean (0/1). |
| **52** | `efm_sx_ib_enabled` | Enables InfiniBand mode. Boolean. |
| **53** | `efm_sx_eth_enabled` | Enables Ethernet mode. Boolean. |
| **54** | `efm_sx_gw_ports` | Number of gateway ports enabled for IB-to-Ethernet bridging. |
| **55** | `efm_sx_max_ufm_ports` | Maximum ports manageable by Mellanox UFM (Unified Fabric Manager). Relevant for enterprise fabric management integration. |
| **56** | `efm_sx_ib_speed_sw_limit` | Software-enforced speed limit on InfiniBand ports (e.g., restrict QDR-capable ports to SDR). Value is the speed cap. |
| **57** | `efm_sx_eth_speed_sw_limit` | Software-enforced speed limit on Ethernet ports (e.g., cap 40GbE ports to 10GbE). Value is the speed cap. |
| **58** | `efm_sx_l3_enabled` | Enables Layer 3 routing features—static routes, OSPF, BGP, etc. Boolean. |
| **59** | `efm_sx_fcf_enabled` | Enables Fibre Channel Forwarder mode for FCoE (Fibre Channel over Ethernet) bridging. Boolean. |

### Option 56 — InfiniBand Speed Limit (`efm_sx_ib_speed_sw_limit`)
| Value | Speed |
|-------|-------|
| 1 | SDR (2.5 Gbps) |
| 2 | DDR (5 Gbps) |
| 4 | QDR (10 Gbps) |
| 8 | FDR10 (10.3 Gbps) |
| 16 | FDR (14 Gbps) |

The SX6036 maxes out at FDR (56 Gbps per port using 4x lanes), so values above 16 wouldn't apply.

### Option 57 — Ethernet Speed Limit (`efm_sx_eth_speed_sw_limit`)
| Value | Speed |
|-------|-------|
| 1 | 1 GbE |
| 10 | 10 GbE |
| 40 | 40 GbE |
| 56 | 56 GbE |

**Note:** These are *limit* options—they restrict maximum speed. For a "no restrictions" license, you'd either:
- Omit these options entirely, or
- Set them to the maximum (16 for IB, 56 for Eth)
- [Since MLNX-OS 3.4.3002, 56GbE speed is enabled by default](https://enterprise-support.nvidia.com/s/article/MLNX2-117-1804kn)

---

To get Ethernet working you'll care about **51** (L2), **53** (Ethernet mode), and **58** (L3 if you want routing). You will also care about **55** if you want to enable VPI mode (Ethernet/Infiniband at the same time).

To install basically all features run this:
```
mellanox-sx6036 [standalone: master] > enable
mellanox-sx6036 [standalone: master] # configure terminal 
mellanox-sx6036 [standalone: master] (config) # license install LK2-EFM_SX-5K11-5L11-5M11-5N21-45P2-145T-115U-1188-A0B4-TJ0Q-0XY
mellanox-sx6036 [standalone: master] (config) # exit
mellanox-sx6036 [standalone: master] # show licenses
```

# Noise and Power
My SX6036 idles at 35w with one PSU plugged in and fans spun down to roughly 4300 rpm (17% speed - any lower and they'll start to rpm cycle). By default the switch idles at 50-55w and the fans run at nearly 10000 rpm.

From an elevated SSH session you can see and control the fans
```
mellanox-sx6036 [standalone: master] > enable
mellanox-sx6036 [standalone: master] # show fan
=====================================================
Module          Device          Fan  Speed     Status
                                     (RPM)
=====================================================
FAN             FAN             F1   9510.00   OK
FAN             FAN             F2   9090.00   OK
FAN             FAN             F3   9510.00   OK
FAN             FAN             F4   9090.00   OK
PS1             FAN             -    -         NOT PRESENT
PS2             FAN             F1   9510.00   OK
mellanox-sx6036 [standalone: master] # fae mlxi2c set_fan /PS1/FAN 1 17
mellanox-sx6036 [standalone: master] # fae mlxi2c set_fan /PS2/FAN 1 17
mellanox-sx6036 [standalone: master] # fae mlxi2c set_fan /FAN/FAN 1 17
mellanox-sx6036 [standalone: master] # show fan
=====================================================
Module          Device          Fan  Speed     Status
                                     (RPM)
=====================================================
FAN             FAN             F1   4290.00   OK
FAN             FAN             F2   4290.00   OK
FAN             FAN             F3   4320.00   OK
FAN             FAN             F4   4290.00   OK
PS1             FAN             -    -         NOT PRESENT
PS2             FAN             F1   4320.00   OK
```

I'll update this when I figure out how to make the fan speed persistent.

# Cable Breakout info
You can configure QSFP ports to breakout to either 2x10GbE SFP+ or 4x10GbE SFP+.
```
mellanox-sx6036 [standalone: master] > enable
mellanox-sx6036 [standalone: master] # configure terminal
mellanox-sx6036 [standalone: master] (config) # interface ethernet 1/36
mellanox-sx6036 [standalone: master] (config interface ethernet 1/36) # shutdown
mellanox-sx6036 [standalone: master] (config interface ethernet 1/36) # exit
mellanox-sx6036 [standalone: master] (config) # interface ethernet 1/33
mellanox-sx6036 [standalone: master] (config interface ethernet 1/33) # shutdown
mellanox-SX6036 [standalone: master] (config interface ethernet 1/33) # module-type qsfp-split-4
the following interfaces will be unmapped: 1/33 1/36 
Type 'yes' to confirm split: yes
mellanox-sx6036 [standalone: master] (config interface ethernet 1/33) # exit
ellanox-SX6036 [standalone: master] (config) # exit
Mellanox-SX6036 [standalone: master] # write memory
```

### SX6036 Port Splitting Options

**Maximum 10 Gb/s Ethernet ports configurable: 64**

<img width="724" height="61" alt="image" src="https://github.com/user-attachments/assets/aed88470-5cb8-4a8f-8b68-3a72e2a842e2" />

#### Port Splitting Table

| Port # | Can be split to 4 | Can be split to 2 |
|--------|-------------------|-------------------|
| 1      | No                | No                |
| 2      | No                | No                |
| 3      | No                | No                |
| 4      | Yes, disables port 1 | Yes            |
| 5      | Yes, disables port 6 | Yes            |
| 6      | No                | No                |
| 7      | No                | No                |
| 8      | No                | No                |
| 9      | Yes, disables port 12 | Yes           |
| 10     | Yes, disables port 7 | Yes            |
| 11     | No                | No                |
| 12     | No                | No                |
| 13     | No                | No                |
| 14     | Yes, disables port 13 | —             |
| 15     | Yes, disables port 18 | —             |
| 16     | No                | No                |
| 17     | No                | No                |
| 18     | No                | No                |
| 19     | No                | No                |
| 20     | No                | Yes               |
| 21     | No                | Yes               |
| 22     | Yes, disables port 19 | Yes           |
| 23     | Yes, disables port 24 | Yes           |
| 24     | No                | No                |
| 25     | No                | No                |
| 26     | No                | No                |
| 27     | Yes, disables port 30 | Yes           |
| 28     | Yes, disables port 25 | Yes           |
| 29     | No                | No                |
| 30     | No                | No                |
| 31     | No                | No                |
| 32     | Yes, disables port 31 | Yes           |
| 33     | Yes, disables port 36 | Yes           |
| 34     | No                | Yes               |
| 35     | No                | Yes               |
| 36     | No                | No                |
