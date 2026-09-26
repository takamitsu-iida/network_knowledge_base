# Catalyst 9000v IOS-XE 17 固有情報

Cisco Modeling Labs 2.10でEVPN VXLAN Layer 3 Overlay Networkの動作検証を行いましたが、動作しませんでした。

[CMLのラボファイル](./02_iosxe_v17_evpn_vxlan_layer3_overlay.md)

ルータの設定はCMLのラボファイルに含まれます。

## `show l2vpn evpn summary`

```
c9kv-1#show bgp l2vpn evpn summary
BGP router identifier 192.168.255.1, local AS number 65000
BGP table version is 1, main routing table version 1
2 network entries using 784 bytes of memory
2 path entries using 464 bytes of memory
1/0 BGP path/bestpath attribute entries using 304 bytes of memory
2 BGP extended community entries using 64 bytes of memory
0 BGP route-map cache entries using 0 bytes of memory
0 BGP filter-list cache entries using 0 bytes of memory
BGP using 1616 total bytes of memory
BGP activity 4/0 prefixes, 4/0 paths, scan interval 60 secs
2 networks peaked at 19:22:20 Sep 26 2026 JST (00:00:08.493 ago)

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.255.3   4        65000       0       0        1    0    0 never    Idle
```

## `show l2vpn evpn route-type 5`
自分の経路のみ。相手からはroute-type 5経路が来ません。
```
c9kv-1#show bgp l2vpn evpn route-type 5
BGP routing table entry for [5][192.168.255.1:1][0][24][10.0.10.0]/17, version 2
Paths: (1 available, best #1, table EVPN-BGP-Table)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local, imported path from base
    0.0.0.0 (via vrf Green) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, metric 0, localpref 100, weight 32768, valid, external, best
      EVPN ESI: 00000000000000000000, Gateway Address: 0.0.0.0, local vtep: 192.168.255.1, VNI Label 50200, MPLS VPN Label 16
      Extended Community: ENCAP:8 Router MAC:5254.0057.A7FE
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 26 2026 19:22:34 JST
BGP routing table entry for [5][192.168.255.1:1][0][64][FD00:1::]/29, version 3
Paths: (1 available, best #1, table EVPN-BGP-Table)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local, imported path from base
    :: (via vrf Green) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, metric 0, localpref 100, weight 32768, valid, external, best
      EVPN ESI: 00000000000000000000, Gateway Address: ::, local vtep: 192.168.255.1, VNI Label 50200, MPLS VPN Label 17
      Extended Community: ENCAP:8 Router MAC:5254.0057.A7FE
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 26 2026 19:22:34 JST
```

## `show nve peers`
route-type 5の経路がないため、VXLANの動的ピアが確立されません。

```
c9kv-1#show nve peers
'M' - MAC entry download flag  'A' - Adjacency download flag
'4' - IPv4 flag  '6' - IPv6 flag

Interface  VNI      Type Peer-IP          RMAC/Num_RTs   eVNI     state flags UP time
```

## `show l2vpn evpn capabilities`
おそらく実装されていません。

* Multi-homing IRB: not supported
    * このプラットフォームでは SVI を介した VXLAN 統合ルーティング（IRB）のデータパス処理機能が無効化または未実装 になっています。

* Ingress replication type: not supported
    * BGP EVPN で Type-5 経路をトリガーにして自動的に Ingress Replication トンネル（L3VNI 用）を動的に生成する機能がプラットフォーム側で弾かれてしまっています。

```
c9kv-1#show l2vpn evpn capabilities
EVPN Platform Capabilities
 VLAN-based EVPN Instance: supported
 VLAN-bundle EVPN Instance: not supported
 VLAN-aware EVPN Instance: not supported
 Ingress replication type: not supported
 Point-to-multipoint replication type: not supported
 Multipoint-to-multipoint replication type: not supported
 Static replication type: supported
 Per-BD MPLS label allocation mode: not supported
 Per-CE MPLS label allocation mode: not supported
 Per-EVI MPLS label allocation mode: not supported
 Address resolution flooding suppression: supported
 DHCP Relay flooding suppression: not supported
 VLAN configuration mode: supported
 MPLS encapsulation: not supported
 VxLAN encapsulation: supported
 Multi-homing aliasing: supported
 Multi-homing IRB: not supported
 VPLS stitching: not supported
 VPLS seamless integration: not supported
 Multi-homing all active redundancy mode: not supported
 Multi-homing single active redundancy mode: supported
 Ethernet Segment old config model: not supported
 IP local learning: supported
 VPLS stitching single-active dual-homing: not supported
 Layer 2 Tenant Routed Multicast IPv4: supported
 Layer 2 Tenant Routed Multicast IPv6: supported
 Layer 2 multicast source specific forwarding: not supported
 VPWS Preferred Path SRTE Policy: not supported
 Multi-homing device ID: not supported
```
