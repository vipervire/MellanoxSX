Here lies info on enabling features on your new (and very old) Mellanox SX6036 switch.

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

Here's what each informational option ID controls:

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

---

To get Ethernet working you'll care about **51** (L2), **53** (Ethernet mode), and **58** (L3 if you want routing). You will also care about **55** if you want to enable VPI mode (Ethernet/Infiniband at the same time).
