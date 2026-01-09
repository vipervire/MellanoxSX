Here lies info on enabling features on your new (and very old) Mellanox SX6036 switch.

## Shell Access
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
- Since MLNX-OS 3.4.3002, 56GbE speed is enabled by default

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

## Noise and Power
My SX6036 idles at 35w with one PSU plugged in and fans spun down to roughly 4300 rpm (17% speed). By default the switch idles at 50-55w and the fans run at nearly 10000 rpm.

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
