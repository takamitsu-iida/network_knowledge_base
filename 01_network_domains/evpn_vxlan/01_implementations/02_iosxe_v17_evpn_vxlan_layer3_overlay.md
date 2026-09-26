---
title: EVPN VXLAN Layer 3 Overlay Network
topic: EVPN VXLAN / Cisco IOS-XE 17.18 / Layer 3 Overlay Network
target_platform: Catalyst 9000v (C9KV)
os_version: Cisco IOS-XE 17.18
keywords:
  - EVPN
  - VXLAN
  - Catalyst 9000v
  - C9KV
  - EVI
  - VNI
  - BGP EVPN
  - IOS-XE 17.18
  - Layer 3 Overlay
updated_at: 2026-09-21
---

# EVPN VXLAN Layer 3 Overlay Network

## EVPN VXLAN Layer 3オーバーレイネットワークとは
EVPN VXLANのLayer 3オーバーレイネットワークを使えば、異なるLayer 2ネットワーク（異なるサブネット）にいるホスト同士でも、Layer 3のルーティング（通信）が可能になります。このとき、ネットワークは「Layer 3 VNI」と「IP VRF」という仕組みを使って、パケットのルーティングを制御します。

本書ではLayer 3オーバーレイネットワークの設定方法のみを説明しています。Layer 2とLayer 3のネットワークを組み合わせて、ルーティングとブリッジングの両方を一つのデバイスで行う「IRB（Integrated Routing and Bridging）」については別の資料を参照してください。

## 交換される経路

### 1. Route Type 5 (IP Prefix Route) - 【主役】
L3オーバーレイの要となるのがこのタイプです。

- 何を伝えるか: サブネット全体（IPプレフィックス）の情報
  - 例：「ネットワーク 10.1.10.0/24 に到達したければ、私のVTEPに送ってくれ」
  - この時、そのサブネットがどの L3 VNI に属しているかという情報も一緒に伝えます。

- 役割: 異なるサブネット間でのルーティングを可能にします。物理ルーターが持つ「ルーティングテーブル」の内容をBGPで全スイッチに配布しているイメージです。


### 2. Route Type 2 (MAC/IP Advertisement Route) - 【IP情報の解決】
L3オーバーレイにおいて、このルートタイプは「個別のホストのIP」を伝えるために使われます。

- 何を伝えるか:
  - 特定のIPアドレス（例: 10.1.10.5/32）
  - そのIPに紐づくMACアドレス
  - そのホストがいるVTEPのIP
- 役割:
  - 「ARP抑制」のために使われます。通信相手のIPアドレスがどのMACアドレス（どのVTEP）にいるかを事前に知っておくことで、無駄なARPブロードキャストを排除します。
  - L3ルーティングの際、宛先IPへの「ラストワンマイル（最後のスイッチからホストへの配送）」を正確に行うために必要です。

## Configuring an IP VRF on a VTEP

### Step 1
```
enable
```

### Example
```
Device> enable
```

### Purpose
Enables privileged EXEC mode. Enter your password, if prompted.

### Step 2
```
configure terminal
```

### Example
```
Device# configure terminal
```

### Purpose
Enters global configuration mode.

Step 3

vrf definition vrf-name

Example:
Device(config)# vrf definition Green
Enters the VRF configuration mode for the specified VRF instance.

Step 4

rd vpn-route-distinguisher

Example:
Device(config-vrf)# rd 100:1
Specifies the route distinguisher for the VRF instance.

Step 5

address-family ipv4 [ multicast | unicast]

Example:
Device(config-vrf)# address-family ipv4
Enters the IPv4 address family configuration mode.

Step 6

route-target { export | import | both} route-target-ext-community

Example:
Device(config-vrf-af)# route-target export 100:1
Example:
Device(config-vrf-af)# route-target import 100:1
Creates a list of import, export, or both import and export route target communities for the specified VRF.

Enter either an autonomous system number and an arbitrary number (xxx:y), or an IP address and an arbitrary number (A.B.C.D:y).

Step 7

route-target { export | import | both} route-target-ext-community stitching

Example:
Device(config-vrf-af)# route-target export 100:1 stitching
Example:
Device(config-vrf-af)# route-target import 100:1 stitching
Configures importing, exporting, or both importing and exporting of EVPN route target communities for the VRF.

Step 8

exit-address-family

Example:
Device(config-vrf-af)# exit-address-family
Exits VRF address family configuration mode and enters VRF configuration mode.

Step 9

address-family ipv6 [ multicast | unicast]

Example:
Device(config-vrf)# address-family ipv6
Enters the IPv6 address family configuration mode.

Step 10

route-target { export | import | both} route-target-ext-community

Example:
Device(config-vrf-af)# route-target export 100:1
Example:
Device(config-vrf-af)# route-target import 100:1
Creates a list of import, export, or both import and export route target communities for the specified VRF.

Enter either an autonomous system number and an arbitrary number (xxx:y), or an IP address and an arbitrary number (A.B.C.D:y).

Step 11

route-target { export | import | both} route-target-ext-community stitching

Example:
Device(config-vrf-af)# route-target export 100:1 stitching
Example:
Device(config-vrf-af)# route-target import 100:1 stitching
Configures importing, exporting, or both importing and exporting of VXLAN route target communities for the VRF.

Step 12

exit-address-family

Example:
Device(config-vrf-af)# exit-address-family
Exits VRF address family configuration mode and enters VRF configuration mode.

Step 13

end

Example:
Device(config-vrf)# end