# Catalyst 9000v IOS-XE 17 固有情報
以下の条件における実装情報です。

- プラットフォーム Catalyst 9000v
- ソフトウェア IOS-XE 17

CML2.10にて検証した結果 Layer 2 EVPN VXLAN は動作しました。CMLラボ内ではMTUは4000バイトに統一しています（設計の推奨値とは異なりますが、CMLの検証環境においては4000バイト程度に抑える必要があるためです）。

[CMLのラボファイル](./00_iosxe_v17_evpn_vxlan.yaml)

## マルチキャストルーティング
BUM通信の扱いは static しかサポートされません。マルチキャストルーティングが必要です。

## ESI
実機で確認したログを以下に示します。

```
c9kv-1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
c9kv-1(config)#l2vpn evpn ethernet-segment ?
  <1-64511>  Ethernet segment local discriminator value

c9kv-1(config)#l2vpn evpn ethernet-segment 10
c9kv-1(config-evpn-es)#?
L2VPN EVPN Ethernet Segment configuration commands:
  default      Set a command to its defaults
  df-election  Designated forwarder election parameters
  exit         Exit from L2VPN evpn Ethernet segment configuration mode
  identifier   Ethernet Segment Identifier
  no           Negate a command or set its defaults
  redundancy   Multi-homing redundancy parameters

c9kv-1(config-evpn-es)#identifier type ?
  0  Type 0 (arbitrary 9-octet ESI value)
  3  Type 3 (MAC-based ESI value)

c9kv-1(config-evpn-es)#identifier type 3 ?
  system-mac  System MAC address for generating the ESI value

c9kv-1(config-evpn-es)#identifier type 3 system-mac ?
  H.H.H  MAC address

c9kv-1(config-evpn-es)#identifier type 3 system-mac 0000.0000.0001 ?
  <cr>  <cr>

c9kv-1(config-evpn-es)#identifier type 3 system-mac 0000.0000.0001
```

`l2vpn evpn ethernet-segment` に続く数字は装置ローカルの識別番号で、ESI値ではありません。

C9Kv(IOS-XE 17.18)で実装しているESIはType 0とType 3です。

Type 3はlocal discriminator値を設定できません。


## VNI
実機で確認したログを以下に示します。

```
c9kv-1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
c9kv-1(config)#interface nve 1
c9kv-1(config-if)#member ?
  vni  Configure VNI information

c9kv-1(config-if)#member vni ?
  WORD  VNI range or instance between 4096-16777215 example: 6010-6030 or 7115
```

VNIの値は4096-16777215になります。


## EVI
実機で確認したログを以下に示します。

```
c9kv-1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
c9kv-1(config)#l2vpn evpn ?
  ethernet-segment  Ethernet segment
  instance          EVPN instance (EVI)
  logging           Configure logging flags
  profile           EVPN Service Profile
  <cr>              <cr>

c9kv-1(config)#l2vpn evpn ins
c9kv-1(config)#l2vpn evpn instance ?
  <1-65535>  EVPN instance identifier value

c9kv-1(config)#l2vpn evpn instance
```

EVIの値は1-65535の範囲で指定します。

## マルチホーミング
実機で確認したログを以下に示します。

```
c9kv-3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
c9kv-3(config)#l2vpn evpn eth
c9kv-3(config)#l2vpn evpn ethernet-segment 20
c9kv-3(config-evpn-es)#redundancy ?
  single-active  Per-vlan load-balancing between PEs on same Ethernet Segment
```

`single-active` だけが実装されており、`all-active`は実装されていません。

## MTU設定
EVPN VXLANのアンダーレイネットワークではMTUを大きくする必要があります。

実機で確認したログを以下に示します。

```
c9kv-3#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
c9kv-3(config)#system mt
c9kv-3(config)#system mtu ?
  <1500-8978>  MTU size in bytes
```

入力できる最大は8978ですがCisco Modeling Labsの仮想環境においては4000バイト程度に抑えておかないと、通信できなくなることがあります。


## スケーラビリティ
Catalyst9000シリーズは機種ごとにマニュアルが分かれています。マニュアルに "Chapter: BGP EVPN VXLAN Scalability Guide" という章があり、そこに最大値が記載されています。

9300シリーズおよび9500シリーズはSDMテンプレートの設定によってサポートされる最大値は変わります。機種よっても違います。どの機種で、どのSDMテンプレートを使うか、を確認する必要があります。

また、サポートされていない機能についても、同じ章に記載されています。

### 9300シリーズ

[BGP EVPN VXLAN Configuration Guide, Cisco IOS XE 26.x.x (Catalyst 9300 Switches)](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/26-x/configuration_guide/vxlan/26x-bgp-evpn-vxlan-9300-cg/scale_and_performance_capabilities_for_bgp_evpn_vxlan.html)

### 9500シリーズ

[BGP EVPN VXLAN Configuration Guide, Cisco IOS XE 17.18.x (Catalyst 9500 Switches)](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9500/software/release/17-18/configuration_guide/vxlan/b_1718_bgp_evpn_vxlan_9500_cg/scale_and_performance_capabilities_for_bgp_evpn_vxlan.html)

Cisco Catalyst 9500X Series switches don't support the following EVPN features in this release

- EVPN VXLAN Aware Flexible NetFlow
- Layer 2 VNI (L2VNI) Multicast Replication BUM Rate-Limiter
- Private VLAN: Community VLAN to L2VNI
- Private VLAN: Isolated VLAN to L2VNI
- EVPN to Layer 2 Handoff: IEEE 802.1ad (QinQ)
- EVPN to MPLS Layer 3 VRF Unicast Handoff
- EVPN to MPLS Layer 3 VRF Multicast Handoff
- EVPN to VPLS Layer 2 Handoff
- EVPN to VPLS Layer 2 Handoff: Neighbors Per VFI
- EVPN to VPLS Layer 2 Handoff: Pseudowire


## 動作確認コマンド

### `show l2vpn evpn evi [ detail]`
EVPNインスタンスの詳細情報を表示します。

実行例。

```
c9kv-1#show l2vpn evpn evi
EVI   VLAN  Ether Tag  L2 VNI    Multicast     Pseudoport
----- ----- ---------- --------- ------------- ------------------
10    10    0          10010     239.1.1.10    Gi1/0/3:10
```

実行例。

```
c9kv-1#show l2vpn evpn evi detail
EVPN instance:          10 (VLAN Based)
  RD:                   192.168.255.1:10 (cfg)
  Import-RTs:           65000:10
  Export-RTs:           65000:10
  Per-EVI Label:        none
  State:                Established
  Replication Type:     Static
  Encapsulation:        vxlan
  Multihoming Aliasing: Enabled (global)
  IP Local Learn:       Enabled (global)
  Adv. Def. Gateway:    Disabled (global)
  Re-originate RT5:     Disabled (global)
  Adv. Multicast:       Disabled (global)
  AR Flood Suppress:    Enabled (global)
  Adv. MAC Only:        Enabled (global)
  Vlan:                 10
    Protected:          False
    Ethernet-Tag:       0
    State:              Established
    Flood Suppress:     Attached
    Core If:
    Access If:
    NVE If:             nve1
    RMAC:               0000.0000.0000
    Core Vlan:          0
    L2 VNI:             10010
    L3 VNI:             0
    VTEP IP:            192.168.254.1
    Originating Router: 192.168.255.1
    MCAST IP:           239.1.1.10
    Pseudoports:
      GigabitEthernet1/0/3 service instance 10 (DF state: forwarding)
        Routes: 1 MAC, 0 MAC/IP
        ESI: 0000.0000.0000.0000.0001
    Peers:
      192.168.254.2
        Routes: 0 MAC, 0 MAC/IP, 0 IMET, 1 EAD, 0 SMET, 0 JOIN-SYNC, 0
LEAVE-SYNC
      192.168.254.3
        Routes: 1 MAC, 0 MAC/IP, 0 IMET, 1 EAD, 0 SMET, 0 JOIN-SYNC, 0
LEAVE-SYNC
      192.168.254.4
        Routes: 0 MAC, 0 MAC/IP, 0 IMET, 1 EAD, 0 SMET, 0 JOIN-SYNC, 0
LEAVE-SYNC
```

### `show l2vpn evpn mac [ detail]`
レイヤ 2 EVPN の MAC アドレスデータベースを表示します。

実行例。

```
c9kv-1#show l2vpn evpn mac
MAC Address    EVI   VLAN  ESI                      Ether Tag  Next Hop(s)
-------------- ----- ----- ------------------------ ---------- ---------------
5254.0071.55a5 10    10    0000.0000.0000.0000.0002 0          192.168.254.3
5254.00d7.5299 10    10    0000.0000.0000.0000.0001 0          Gi1/0/3:10

c9kv-1#show l2vpn evpn mac det
c9kv-1#show l2vpn evpn mac detail
MAC Address:                5254.0071.55a5
EVPN Instance:              10
Vlan:                       10
Ethernet Segment:           0000.0000.0000.0000.0002
Ethernet Tag ID:            0
Next Hop(s):                V:10010 192.168.254.3
Local Address:              192.168.254.1
Sequence Number:            0
MAC only present:           Yes
MAC Duplication Detection:  Timer not running

MAC Address:                5254.00d7.5299
EVPN Instance:              10
Vlan:                       10
Ethernet Segment:           0000.0000.0000.0000.0001
Ethernet Tag ID:            0
Next Hop(s):                V:10010 GigabitEthernet1/0/3 service instance 10
Sequence Number:            0
MAC only present:           Yes
MAC Duplication Detection:  Timer not running
```



### `show l2vpn evpn summary`
レイヤ2 EVPN 情報の要旨を表示します。

実行例。

```
c9kv-1#show l2vpn evpn summary
L2VPN EVPN
  EVPN Instances (excluding point-to-point): 1
    VLAN Based:   1
  Vlans: 1
  BGP: ASN 65000, address-family l2vpn evpn configured
  Router ID: 192.168.255.1
  Global Replication Type: Not set
  ARP/ND Flooding Suppression: Enabled
  Connectivity to Core: UP
  BGP core status: UP
  Core link status: N/A
  MAC Duplication: seconds 180 limit 5
  MAC Addresses: 2
    Local:     1
    Remote:    1
    Duplicate: 0
  IP Duplication: seconds 180 limit 5
  IP Addresses: 0
    Local:     0
    Remote:    0
    Duplicate: 0
  Advertise Default Gateway: No
  Default Gateway Addresses: 0
   Local:      0
   Remote:     0
  Maximum number of Route Targets per EAD-ES route: 200
  Multi-home aliasing: Enabled
  IP local learning tracking: Disabled
  Global IP Local Learn: Enabled
  IP local learning limits
    IPv4: 4 addresses per-MAC
    IPv6: 12 addresses per-MAC
  IP local learning timers
    Down:      10 minutes
    Poll:      1 minutes
    Reachable: 5 minutes
    Stale:     30 minutes
  Auto route-target: evi-id based
  Advertise Multicast: No
  Global Anycast Gateway MAC: No
  MAC Only Advertisement: Enabled
```

### `show l2vpn evpn capabilities`
レイヤ 2 EVPN のプラットフォーム機能情報を表示します。

実行例。

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

### `show l2vpn evpn peers`
レイヤ 2 EVPN ピアルートカウントと稼働時間を表示します。

実行例。

```
c9kv-1#show l2vpn evpn peers ?
  vxlan  VxLAN peers

c9kv-1#show l2vpn evpn peers vx
c9kv-1#show l2vpn evpn peers vxlan ?
  address    Peer IP address
  detail     Detailed output
  global     Routes global to all EVPN instances
  interface  NVE interface
  vni        VxLAN network identifier
  |          Output modifiers
  <cr>       <cr>

c9kv-1#show l2vpn evpn peers vxlan

Interface VNI      Peer-IP                                 Num routes
eVNI     UP time
--------- -------- --------------------------------------- ------------------ --------
Global    N/A      192.168.254.2                           2
N/A      04:20:11
Global    N/A      192.168.254.3                           2
N/A      04:20:11
Global    N/A      192.168.254.4                           2
N/A      04:20:11
nve1      10010    192.168.254.2                           1
10010    04:20:11
nve1      10010    192.168.254.3                           2
10010    04:20:11
nve1      10010    192.168.254.4                           1
10010    04:20:11
```

### `show l2vpn evpn route-target`
レイヤ 2 EVPN インポートルートのターゲットを表示します。

実行例。

```
c9kv-1#show l2vpn evpn route-target
Route Target           EVPN Instances
65000:10               10
```

### `show l2vpn evpn memory`
レイヤ 2 EVPN メモリの使用量を表示します。

実行例。

```
c9kv-1#show l2vpn evpn memory
  Allocator-Name                  In-use/Allocated            Count
  ----------------------------------------------------------------------------
  EVPN BD EFP member chunk  :         24/1592       (  1%) [      1] Chunk
  EVPN BD EVI member chunk  :         24/1592       (  1%) [      1] Chunk
  EVPN DB                   :        720/65576      (  1%) [     10] Chunk
  EVPN DM MANAGER CHUNK     :          0/69984      (  0%) [      0] Chunk
  EVPN Ether Seg chunk      :         32/1592       (  2%) [      1] Chunk
  EVPN MAC Address Info chu :        240/131264     (  0%) [      2] Chunk
  EVPN MAC remote NH        :        172/264        ( 65%) [      1]
  EVPN MGR DB               :       2232/65576      (  3%) [     31] Chunk
  EVPN MLRIB MGR            :        316/408        ( 77%) [      1]
  EVPN Mgr BD PP chunk      :        216/7504       (  2%) [      1] Chunk
  EVPN Mgr Bucket chunk     :         32/1592       (  2%) [      1] Chunk
  EVPN Mgr EFI chunk        :        592/20096      (  2%) [      1] Chunk
  EVPN Mgr EVI chunk        :        288/10096      (  2%) [      1] Chunk
  EVPN Mgr Eth seg chunk    :        720/16680      (  4%) [      2] Chunk
  EVPN Mgr Fwdr chunk       :        224/2976       (  7%) [      4] Chunk
  EVPN Mgr Msg chunk        :          0/10096      (  0%) [      0] Chunk
  EVPN Mgr Peer chunk       :        576/4392       ( 13%) [      6] Chunk
  EVPN Mgr RT chunk         :         24/1592       (  1%) [      1] Chunk
  EVPN Mgr Thread           :     589928/1352424    ( 43%) [   8288]
  EVPN Mgr access i/f chunk :         24/1592       (  1%) [      1] Chunk
  EVPN VPWS Thread          :       1908/2368       ( 80%) [      5]
  EVPN dtrace stridx        :    1194876/1194968    ( 99%) [      1]
  EVPN dtrace stridx freeli :     132764/132856     ( 99%) [      1]
  EVPN dtrace stridx hash   :         52/144        ( 36%) [      1]
  EVPN dtrace stridx slots  :     265532/265624     ( 99%) [      1]
  EVPN dtrace stridx2slot   :     132764/132856     ( 99%) [      1]
  EVPN instance chunk       :        208/7504       (  2%) [      1] Chunk
  EVPN main thread          :       2576/3312       ( 77%) [      8]
  EVPN rt-db ee             :         52/144        ( 36%) [      1]
  EVPN rt-db rte            :        116/208        ( 55%) [      1]

  Total allocated: 3.343 Mb, 3424 Kb, 3506872 bytes
```

### `show l2route evpn summary`
EVPN ルートの要旨を表示します。

実行例。

```
c9kv-1#show l2route evpn summary
Object         Static      BGP    L2VPN      Total
------------ -------- -------- -------- ----------
TOPOLOGY            0        0        1          1
MAC                 0        1        1          2
IMET                1        0        0          1
ES                  0        3        1          4
EAD per-EVI         0        2        1          3
EAD per-ES          0        2        1          3
MAC_IP              0        0        0          0
Peer                0        6        0          6
DG                  0        0        0          0
SMET                0        0        0          0
MC-ROUTE            0        0        0          0
MC-JOIN-SYNC        0        0        0          0
MC-LEAVE-SYN        0        0        0          0
Total               1       14        4         19
Valid Clients Bitmap: 00000007
```

### `show l2route evpn mac [ detail]`
EVPN コントロールプレーンでスイッチが学習した MAC アドレス情報を表示します。

実行例。

```
c9kv-1#show l2route evpn mac
  EVI       ETag   Prod    Mac Address
         Next Hop(s) Seq Number
----- ---------- ------ --------------
---------------------------------------------------- ----------
   10          0    BGP 5254.0071.55a5
V:10010 192.168.254.3          0
   10          0  L2VPN 5254.00d7.5299
          Gi1/0/3:10          0

c9kv-1#show l2route evpn mac detail
EVPN Instance:            10
Ethernet Tag:             0
Producer Name:            BGP
MAC Address:              5254.0071.55a5
Num of MAC IP Route(s):   0
Sequence Number:          0
ESI:                      0000.0000.0000.0000.0002
Flags:                    B()
Next Hop(s):              V:10010 192.168.254.3
MAC Orig Next Hop(s):     V:10010 192.168.254.3
Resolved Next Hops:       V:10010 192.168.254.3, V:10010 192.168.254.4
Resolved Redundancy Mode: Single-Active

EVPN Instance:            10
Ethernet Tag:             0
Producer Name:            L2VPN
MAC Address:              5254.00d7.5299
Num of MAC IP Route(s):   0
Sequence Number:          0
ESI:                      0000.0000.0000.0000.0001
Flags:                    B()
Next Hop(s):              Gi1/0/3:10
```

### `show l2route evpn imet detail`
レイヤ 2 EVPN アドレスファミリの IMET ルートの詳細を表示します。このコマンドは、入力の複製を使用して転送されたトラフィックに関する詳細のみを表示します。

実行例。

```
c9kv-1#show l2route evpn imet detail
EVPN Instance:            10
Ethernet Tag:             0
Producer Name:            Static
Router IP Addr:           192.168.255.1
Route Ethernet Tag:       0
Tunnel Flags:             0
Tunnel Type:              No tunnel information present
Tunnel Labels:            10010
Tunnel ID:                239.1.1.10
Multicast Proxy:          No
Next Hop(s):              N/A
```

### `show bgp l2vpn evpn`
レイヤ 2 VPN EVPN アドレスファミリの BGP 情報を表示します。

実行例。

```
c9kv-1#show bgp l2vpn evpn
BGP table version is 56, local router ID is 192.168.255.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path, L long-lived-stale,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
Route Distinguisher: 192.168.255.1:2
 *>   [1][192.168.255.1:2][00000000000000000001][4294967295]/23
                      0.0.0.0                            32768 ?
Route Distinguisher: 192.168.255.2:2
 *>i  [1][192.168.255.2:2][00000000000000000001][4294967295]/23
                      192.168.254.2            0    100      0 ?
Route Distinguisher: 192.168.255.3:2
 *>i  [1][192.168.255.3:2][00000000000000000002][4294967295]/23
                      192.168.254.3            0    100      0 ?
Route Distinguisher: 192.168.255.4:2
 *>i  [1][192.168.255.4:2][00000000000000000002][4294967295]/23
                      192.168.254.4            0    100      0 ?
Route Distinguisher: 192.168.255.1:10
     Network          Next Hop            Metric LocPrf Weight Path
 *mi  [1][192.168.255.1:10][00000000000000000001][0]/23
                      192.168.254.2            0    100      0 ?
 *>                    0.0.0.0                            32768 ?
 *mi  [1][192.168.255.1:10][00000000000000000002][0]/23
                      192.168.254.4            0    100      0 ?
 *>i                   192.168.254.3            0    100      0 ?
Route Distinguisher: 192.168.255.2:10
 *>i  [1][192.168.255.2:10][00000000000000000001][0]/23
                      192.168.254.2            0    100      0 ?
Route Distinguisher: 192.168.255.3:10
 *>i  [1][192.168.255.3:10][00000000000000000002][0]/23
                      192.168.254.3            0    100      0 ?
Route Distinguisher: 192.168.255.4:10
 *>i  [1][192.168.255.4:10][00000000000000000002][0]/23
                      192.168.254.4            0    100      0 ?
Route Distinguisher: 192.168.255.1:10
 *>i  [2][192.168.255.1:10][0][48][5254007155A5][0][*]/20
                      192.168.254.3            0    100      0 ?
 *>   [2][192.168.255.1:10][0][48][525400D75299][0][*]/20
                      0.0.0.0                            32768 ?
Route Distinguisher: 192.168.255.3:10
 *>i  [2][192.168.255.3:10][0][48][5254007155A5][0][*]/20
     Network          Next Hop            Metric LocPrf Weight Path
                      192.168.254.3            0    100      0 ?
Route Distinguisher: 192.168.255.1:1
 *>   [4][192.168.255.1:1][00000000000000000001][32][192.168.255.1]/23
                      0.0.0.0                            32768 ?
Route Distinguisher: 192.168.255.2:1
 *>i  [4][192.168.255.2:1][00000000000000000001][32][192.168.255.2]/23
                      192.168.254.2            0    100      0 ?
Route Distinguisher: 192.168.255.3:2
 *>i  [4][192.168.255.3:2][00000000000000000002][32][192.168.255.3]/23
                      192.168.254.3            0    100      0 ?
Route Distinguisher: 192.168.255.4:2
 *>i  [4][192.168.255.4:2][00000000000000000002][32][192.168.255.4]/23
                      192.168.254.4            0    100      0 ?
```

### `show bgp l2vpn evpn route-type 2`
L2VPN EVPN アドレスファミリのルートタイプ 2 の BGP 情報を表示します。

実行例。

```
c9kv-1#show bgp l2vpn evpn route-type 2
BGP routing table entry for
[2][192.168.255.1:10][0][48][5254007155A5][0][*]/20, version 56
Paths: (1 available, best #1, table evi_10)
  Not advertised to any peer
  Refresh Epoch 2
  Local, imported path from
[2][192.168.255.3:10][0][48][5254007155A5][0][*]/20 (global)
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:16 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][525400D75299][0][*]/20, version 38
Paths: (1 available, best #1, table evi_10)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      EVPN ESI: 00000000000000000001, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      Local irb vxlan vtep:
        vrf:not found, l3-vni:0
        local router mac:0000.0000.0000
        core-irb interface:(not found)
        vtep-ip:192.168.254.1
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:48 JST
BGP routing table entry for
[2][192.168.255.3:10][0][48][5254007155A5][0][*]/20, version 55
Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:16 JST
```

### `show bgp l2vpn evpn evi context`
レイヤ 2 EVPN インスタンスのコンテキスト情報を表示します。

実行例。

```
c9kv-1#show bgp l2vpn evpn evi context
 EVI                             Default RD              L2TopoCnt
 evi_10                          192.168.255.1:10              1
```

### `show nve vni`
NVE インターフェイスに関連付けられた VXLAN ネットワーク識別子のメンバーに関する情報を表示します。

実行例。

```
c9kv-1#show nve vni
Interface  VNI        Multicast-group  VNI state  Mode  VLAN  cfg vrf
nve1       10010      239.1.1.10       Up         L2CP  10    CLI N/A
```

### `show nve vni <vni-id> detail`
VXLAN ネットワーク識別子のメンバーの詳細な NVE インターフェイスの状態の情報を表示します。

実行例。

```
c9kv-1#show nve vni 10010 detail
Interface  VNI        Multicast-group  VNI state  Mode  VLAN  cfg vrf
nve1       10010      239.1.1.10       Up         L2CP  10    CLI N/A

L2CP VNI IRB state: IPv4 down, IPv6 down
VNI IRB down reason:
BDI if un-configured

L2CP VNI local VTEP info:
VLAN: 10
SVI if handler: 0x0
Local VTEP: 192.168.254.1
Local routing: Disabled

VNI Detailed statistics:
   Pkts In   Bytes In   Pkts Out  Bytes Out
        12       1966         12       2010
```

### `show nve peers`
ピアリーフスイッチの NVE インターフェイスの状態の情報を表示します。

実行例。

```
c9kv-1#show nve peers
'M' - MAC entry download flag  'A' - Adjacency download flag
'4' - IPv4 flag  '6' - IPv6 flag

Interface  VNI      Type Peer-IP          RMAC/Num_RTs   eVNI
state flags UP time
nve1       10010    L2CP 192.168.254.2    1              10010      UP
  N/A  04:28:32
nve1       10010    L2CP 192.168.254.3    4              10010      UP
  N/A  04:28:32
nve1       10010    L2CP 192.168.254.4    1              10010      UP
  N/A  04:28:32
```

### `show mac address-table vlan <vlan-id>`
VLAN の MAC アドレスを表示します。

実行例。

```
c9kv-1#show mac address-table vlan 10
          Mac Address Table
-------------------------------------------

Vlan    Mac Address       Type        Ports
----    -----------       --------    -----
  10    5254.00d7.5299    DYNAMIC     Gi1/0/3
  10    aabb.cc00.0100    DYNAMIC     Gi1/0/3
Total Mac Addresses for this criterion: 2
```

### `show platform software fed switch active matm macTable vlan <vlan-id>`
転送エンジンドライバ（FED）の MAC アドレス テーブル マネージャ データベースから VLAN の MAC アドレスを表示します。

実行例。

```
c9kv-1#show platform software fed switch active matm macTable vlan 10
VLAN   MAC                   Type  Seq#    EC_Bi  Flags  machandle
      siHandle            riHandle            diHandle
*a_time  *e_time  ports
         Con
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
10     5254.0071.55a5   0x1000001       0      0      0  0x0
      0x44a               0x44a               0x0
   0        0  RLOC 192.168.254.3 adj_id 22
      No




10     5254.00d7.5299         0x1       2      0      0  0x0
      0x0                 0x0                 0x0
 300        0  GigabitEthernet1/0/3
      No




10     aabb.cc00.0200   0x1000001       0      0      0  0x0
      0x44a               0x44a               0x0
   0        0  RLOC 192.168.254.3 adj_id 22
      No




10     aabb.cc00.0100         0x1       3      0      0  0x0
      0x0                 0x0                 0x0
 300      119  GigabitEthernet1/0/3
      No





Total Mac number of addresses:: 4
Summary:
Total number of secure addresses:: 0
Total number of drop addresses:: 0
Total number of lisp local addresses:: 0
Total number of lisp remote addresses:: 2
*a_time=aging_time(secs)  *e_time=total_elapsed_time(secs)
Type:
MAT_DYNAMIC_ADDR           0x1  MAT_STATIC_ADDR            0x2
MAT_CPU_ADDR               0x4  MAT_DISCARD_ADDR           0x8
MAT_ALL_VLANS             0x10  MAT_NO_FORWARD            0x20
MAT_IPMULT_ADDR           0x40  MAT_RESYNC                0x80
MAT_DO_NOT_AGE           0x100  MAT_SECURE_ADDR          0x200
MAT_NO_PORT              0x400  MAT_DROP_ADDR            0x800
MAT_DUP_ADDR            0x1000  MAT_NULL_DESTINATION    0x2000
MAT_DOT1X_ADDR          0x4000  MAT_ROUTER_ADDR         0x8000
MAT_WIRELESS_ADDR      0x10000  MAT_SECURE_CFG_ADDR    0x20000
MAT_OPQ_DATA_PRESENT   0x40000  MAT_WIRED_TUNNEL_ADDR  0x80000
MAT_DLR_ADDR          0x100000  MAT_MRP_ADDR          0x200000
MAT_MSRP_ADDR         0x400000  MAT_LISP_LOCAL_ADDR   0x800000
MAT_LISP_REMOTE_ADDR 0x1000000  MAT_VPLS_ADDR        0x2000000
MAT_LISP_GW_ADDR     0x4000000  MAT_ALIASING         0x8000000
MAT_CDP_BYPASS_ADDR 0x10000000
```

### `show device-tracking database`
デバイス トラッキング データベースを表示します。

実行例。

```
c9kv-1#show device-tracking database
Binding Table has 1 entries, 1 dynamic (limit 200000)
Codes: L - Local, S - Static, ND - Neighbor Discovery, ARP - Address
Resolution Protocol, DH4 - IPv4 DHCP, DH6 - IPv6 DHCP, PKT - Other
Packet, API - API created
Preflevel flags (prlvl):
0001:MAC and LLA match     0002:Orig trunk            0004:Orig access
0008:Orig trusted trunk    0010:Orig trusted access   0020:DHCP assigned
0040:Cga authenticated     0080:Cert authenticated    0100:Statically assigned


    Network Layer Address                    Link Layer Address
Interface  vlan       prlvl      age        state      Time left
ARP 10.0.0.1                                 aabb.cc00.0100
Gi1/0/3    10         0005       3mn        REACHABLE  109 s
```

### `show device-tracking database mac`
デバイストラッキング MAC アドレスデータベースを表示します。

実行例。

```
c9kv-1#show device-tracking database mac
 MAC                    Interface  vlan       prlvl      state
   Time left        Policy           Input_index
 aabb.cc00.0100         Gi1/0/3    10         NO TRUST   MAC-REACHABLE
   84 s             evpn-device-track 1033
```

### `show ip mroute`
マルチキャスト ルーティング テーブル情報を表示します。

実行例。

```
c9kv-1#show ip mroute
IP Multicast Routing Table
Flags: D - Dense, S - Sparse, B - Bidir Group, s - SSM Group, C - Connected,
       L - Local, P - Pruned, R - RP-bit set, F - Register flag,
       T - SPT-bit set, J - Join SPT, M - MSDP created entry, E - Extranet,
       X - Proxy Join Timer Running, A - Candidate for MSDP Advertisement,
       U - URD, I - Received Source Specific Host Report,
       Z - Multicast Tunnel, z - MDT-data group sender,
       Y - Joined MDT-data group, y - Sending to MDT-data group,
       G - Received BGP C-Mroute, g - Sent BGP C-Mroute,
       N - Received BGP Shared-Tree Prune, n - BGP C-Mroute suppressed,
       Q - Received BGP S-A Route, q - Sent BGP S-A Route,
       V - RD & Vector, v - Vector, p - PIM Joins on route,
       x - VxLAN group, c - PFP-SA cache created entry,
       * - determined by Assert, # - iif-starg configured on rpf intf,
       e - encap-helper tunnel flag, l - LISP decap ref count contributor
Outgoing interface flags: H - Hardware switched, A - Assert winner, p - PIM Join
                          t - LISP transit group
 Timers: Uptime/Expires
 Interface state: Interface, Next-Hop or VCD, State/Mode

(*, 239.1.1.10), 04:32:53/00:03:15, RP 192.168.255.1, flags: SJCx
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet1/0/1, Forward/Sparse, 04:31:48/00:02:34, flags:
    GigabitEthernet1/0/2, Forward/Sparse, 04:31:58/00:03:15, flags:
    Tunnel0, Forward/Sparse-Dense, 04:32:53/00:00:07, flags:

(*, 224.0.1.40), 04:35:34/00:03:28, RP 192.168.255.1, flags: SJCL
  Incoming interface: Null, RPF nbr 0.0.0.0
  Outgoing interface list:
    GigabitEthernet1/0/1, Forward/Sparse, 04:31:48/00:02:55, flags:
    GigabitEthernet1/0/2, Forward/Sparse, 04:31:58/00:03:28, flags:
    Loopback0, Forward/Sparse, 04:35:33/00:02:29, flags:
```

### `show ip bgp l2vpn evpn detail`
ルートの詳細情報を表示します。

実行例。

```
c9kv-1#show ip bgp l2vpn evpn detail

Route Distinguisher: 192.168.255.1:2
BGP routing table entry for
[1][192.168.255.1:2][00000000000000000001][4294967295]/23, version 23
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      Rcvd Label: None, Local Label: 0, local vtep: 192.168.254.1
      Extended Community: RT:65000:10 ENCAP:8 EVPN LABEL:0x1:Label-0
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:40 JST

Route Distinguisher: 192.168.255.2:2
BGP routing table entry for
[1][192.168.255.2:2][00000000000000000001][4294967295]/23, version 32
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.2 (metric 20) (via default) from 192.168.255.2 (192.168.255.2)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Rcvd Label: 0, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8 EVPN LABEL:0x1:Label-0
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:41 JST

Route Distinguisher: 192.168.255.3:2
BGP routing table entry for
[1][192.168.255.3:2][00000000000000000002][4294967295]/23, version 40
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Rcvd Label: 0, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8 EVPN LABEL:0x1:Label-0
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:08 JST

Route Distinguisher: 192.168.255.4:2
BGP routing table entry for
[1][192.168.255.4:2][00000000000000000002][4294967295]/23, version 48
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.4 (metric 30) (via default) from 192.168.255.4 (192.168.255.4)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Rcvd Label: 0, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8 EVPN LABEL:0x1:Label-0
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:08 JST

Route Distinguisher: 192.168.255.1:10
BGP routing table entry for
[1][192.168.255.1:10][00000000000000000001][0]/23, version 36
  Paths: (2 available, best #2, table evi_10)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local, imported path from
[1][192.168.255.2:10][00000000000000000001][0]/23 (global)
    192.168.254.2 (metric 20) (via default) from 192.168.255.2 (192.168.255.2)
      Origin incomplete, metric 0, localpref 100, valid, internal,
multipath(oldest)
      Rcvd Label: 10010, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0
      Updated on Sep 24 2026 14:19:41 JST
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, multipath, best
      Rcvd Label: None, Local Label: 10010
      Extended Community: RT:65000:10 ENCAP:8
      Local irb vxlan vtep:
        vrf:not found, l3-vni:0
        local router mac:0000.0000.0000
        core-irb interface:(not found)
        vtep-ip:192.168.254.1
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:40 JST
BGP routing table entry for
[1][192.168.255.1:10][00000000000000000002][0]/23, version 53
  Paths: (2 available, best #2, table evi_10)
  Not advertised to any peer
  Refresh Epoch 2
  Local, imported path from
[1][192.168.255.4:10][00000000000000000002][0]/23 (global)
    192.168.254.4 (metric 30) (via default) from 192.168.255.4 (192.168.255.4)
      Origin incomplete, metric 0, localpref 100, valid, internal,
multipath(oldest)
      Rcvd Label: 10010, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0
      Updated on Sep 24 2026 14:23:08 JST
  Refresh Epoch 2
  Local, imported path from
[1][192.168.255.3:10][00000000000000000002][0]/23 (global)
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal,
multipath, best
      Rcvd Label: 10010, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:08 JST

Route Distinguisher: 192.168.255.2:10
BGP routing table entry for
[1][192.168.255.2:10][00000000000000000001][0]/23, version 34
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.2 (metric 20) (via default) from 192.168.255.2 (192.168.255.2)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Rcvd Label: 10010, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:41 JST

Route Distinguisher: 192.168.255.3:10
BGP routing table entry for
[1][192.168.255.3:10][00000000000000000002][0]/23, version 42
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Rcvd Label: 10010, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:08 JST

Route Distinguisher: 192.168.255.4:10
BGP routing table entry for
[1][192.168.255.4:10][00000000000000000002][0]/23, version 51
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.4 (metric 30) (via default) from 192.168.255.4 (192.168.255.4)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Rcvd Label: 10010, Local Label: None
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:08 JST

Route Distinguisher: 192.168.255.1:10
BGP routing table entry for
[2][192.168.255.1:10][0][48][5254007155A5][0][*]/20, version 56
  Paths: (1 available, best #1, table evi_10)
  Not advertised to any peer
  Refresh Epoch 2
  Local, imported path from
[2][192.168.255.3:10][0][48][5254007155A5][0][*]/20 (global)
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:16 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][525400D75299][0][*]/20, version 38
  Paths: (1 available, best #1, table evi_10)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      EVPN ESI: 00000000000000000001, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      Local irb vxlan vtep:
        vrf:not found, l3-vni:0
        local router mac:0000.0000.0000
        core-irb interface:(not found)
        vtep-ip:192.168.254.1
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:48 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][AABBCC000100][0][*]/20, version 62
  Paths: (1 available, best #1, table evi_10)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      EVPN ESI: 00000000000000000001, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      Local irb vxlan vtep:
        vrf:not found, l3-vni:0
        local router mac:0000.0000.0000
        core-irb interface:(not found)
        vtep-ip:192.168.254.1
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:34 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][AABBCC000100][32][10.0.0.1]/24, version
63
  Paths: (1 available, best #1, table evi_10)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      EVPN ESI: 00000000000000000001, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      Local irb vxlan vtep:
        vrf:not found, l3-vni:0
        local router mac:0000.0000.0000
        core-irb interface:(not found)
        vtep-ip:192.168.254.1
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:34 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][AABBCC000200][0][*]/20, version 60
  Paths: (1 available, best #1, table evi_10)
  Not advertised to any peer
  Refresh Epoch 2
  Local, imported path from
[2][192.168.255.3:10][0][48][AABBCC000200][0][*]/20 (global)
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:25 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][AABBCC000200][32][10.0.0.2]/24, version
58
  Paths: (1 available, best #1, table evi_10)
  Not advertised to any peer
  Refresh Epoch 2
  Local, imported path from
[2][192.168.255.3:10][0][48][AABBCC000200][32][10.0.0.2]/24 (global)
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:24 JST

Route Distinguisher: 192.168.255.3:10
BGP routing table entry for
[2][192.168.255.3:10][0][48][5254007155A5][0][*]/20, version 55
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:16 JST
BGP routing table entry for
[2][192.168.255.3:10][0][48][AABBCC000200][0][*]/20, version 59
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:25 JST
BGP routing table entry for
[2][192.168.255.3:10][0][48][AABBCC000200][32][10.0.0.2]/24, version
57
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:25 JST

Route Distinguisher: 192.168.255.1:1
BGP routing table entry for
[4][192.168.255.1:1][00000000000000000001][32][192.168.255.1]/23,
version 26
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      Local vtep: 192.168.254.1
      Extended Community: ENCAP:8 EVPN ES-IMPORT:0x0:0x0:0x0
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:40 JST

Route Distinguisher: 192.168.255.2:1
BGP routing table entry for
[4][192.168.255.2:1][00000000000000000001][32][192.168.255.2]/23,
version 33
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.2 (metric 20) (via default) from 192.168.255.2 (192.168.255.2)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Extended Community: ENCAP:8 EVPN ES-IMPORT:0x0:0x0:0x0
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:41 JST

Route Distinguisher: 192.168.255.3:2
BGP routing table entry for
[4][192.168.255.3:2][00000000000000000002][32][192.168.255.3]/23,
version 41
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Extended Community: ENCAP:8 EVPN ES-IMPORT:0x0:0x0:0x0
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:08 JST

Route Distinguisher: 192.168.255.4:2
BGP routing table entry for
[4][192.168.255.4:2][00000000000000000002][32][192.168.255.4]/23,
version 49
  Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.4 (metric 30) (via default) from 192.168.255.4 (192.168.255.4)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      Extended Community: ENCAP:8 EVPN ES-IMPORT:0x0:0x0:0x0
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:08 JST
```

### `show ip bgp l2vpn evpn route-type 2`
IP アドレスのみ、または MAC アドレスと IP アドレスの両方を含むルートを表示します。

実行例。

```
c9kv-1#show ip bgp l2vpn evpn route-type 2
BGP routing table entry for
[2][192.168.255.1:10][0][48][5254007155A5][0][*]/20, version 56
Paths: (1 available, best #1, table evi_10)
  Not advertised to any peer
  Refresh Epoch 2
  Local, imported path from
[2][192.168.255.3:10][0][48][5254007155A5][0][*]/20 (global)
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:16 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][525400D75299][0][*]/20, version 38
Paths: (1 available, best #1, table evi_10)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      EVPN ESI: 00000000000000000001, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      Local irb vxlan vtep:
        vrf:not found, l3-vni:0
        local router mac:0000.0000.0000
        core-irb interface:(not found)
        vtep-ip:192.168.254.1
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:19:48 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][AABBCC000100][0][*]/20, version 62
Paths: (1 available, best #1, table evi_10)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      EVPN ESI: 00000000000000000001, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      Local irb vxlan vtep:
        vrf:not found, l3-vni:0
        local router mac:0000.0000.0000
        core-irb interface:(not found)
        vtep-ip:192.168.254.1
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:34 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][AABBCC000100][32][10.0.0.1]/24, version
63
Paths: (1 available, best #1, table evi_10)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  Local
    0.0.0.0 (via default) from 0.0.0.0 (192.168.255.1)
      Origin incomplete, localpref 100, weight 32768, valid, sourced,
local, best
      EVPN ESI: 00000000000000000001, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      Local irb vxlan vtep:
        vrf:not found, l3-vni:0
        local router mac:0000.0000.0000
        core-irb interface:(not found)
        vtep-ip:192.168.254.1
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:34 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][AABBCC000200][0][*]/20, version 67
Paths: (1 available, best #1, table evi_10)
  Not advertised to any peer
  Refresh Epoch 2
  Local, imported path from
[2][192.168.255.3:10][0][48][AABBCC000200][0][*]/20 (global)
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:12:58 JST
BGP routing table entry for
[2][192.168.255.1:10][0][48][AABBCC000200][32][10.0.0.2]/24, version
58
Paths: (1 available, best #1, table evi_10)
  Not advertised to any peer
  Refresh Epoch 2
  Local, imported path from
[2][192.168.255.3:10][0][48][AABBCC000200][32][10.0.0.2]/24 (global)
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:24 JST
BGP routing table entry for
[2][192.168.255.3:10][0][48][5254007155A5][0][*]/20, version 55
Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 14:23:16 JST
BGP routing table entry for
[2][192.168.255.3:10][0][48][AABBCC000200][0][*]/20, version 66
Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:12:58 JST
BGP routing table entry for
[2][192.168.255.3:10][0][48][AABBCC000200][32][10.0.0.2]/24, version
57
Paths: (1 available, best #1, table EVPN-BGP-Table)
  Not advertised to any peer
  Refresh Epoch 2
  Local
    192.168.254.3 (metric 20) (via default) from 192.168.255.3 (192.168.255.3)
      Origin incomplete, metric 0, localpref 100, valid, internal, best
      EVPN ESI: 00000000000000000002, Label1 10010
      Extended Community: RT:65000:10 ENCAP:8
      rx pathid: 0, tx pathid: 0x0
      Updated on Sep 24 2026 18:07:25 JST
```
