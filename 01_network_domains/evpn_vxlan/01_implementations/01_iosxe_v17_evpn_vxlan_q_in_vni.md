---
title: EVPN VXLAN Layer 2 Overlay with Q-in-VNI
topic: EVPN VXLAN / Cisco IOS-XE 17.18 / Q-in-VNI
target_platform: Catalyst 9000v (C9KV)
os_version: Cisco IOS-XE 17.18
keywords:
  - EVPN
  - VXLAN
  - ESI
  - Single-Active
  - Catalyst 9000v
  - C9KV
  - EVI
  - VNI
  - BGP EVPN
  - IOS-XE 17.18
updated_at: 2026-09-21
---

# EVPN VXLAN Layer 2 Overlay with Q-in-VNI
BGP EVPN VXLANネットワークにおいて、Layer 2 VNI越しにIEEE 802.1Qタグを運ぶ技術（Q-in-VNI）は、ネットワークの拡張性と分離（アイソレーション）に関する制約を解消するためのものです。この技術では、VXLANヘッダーの中に802.1Qタグをそのまま（透過的に）含めることができます。これにより、複数のVLAN（802.1Qタグ）が混在するトランク接続を、単一のLayer 2 VNIを使ってEVPN VXLANファブリック全体へそのまま中継できるようになります。Q-in-VNIを活用することで、キャンパスネットワークのように膨大な数のLayer 2オーバーレイが必要な環境においても、高い柔軟性と拡張性を維持しながら、より多くの仮想ネットワークを構築することが可能になります。

本設計はベースとなるレイヤ2オーバーレイネットワークが構築済みであることを前提とします。

[Catalyst 9000vにおける Single-Active ESI を用いたEVPN over VXLAN 設定ガイド](./00_iosxe_v17_evpn_config.md)

## Q-in-VNI
企業キャンパス、データセンター、サービスプロバイダーのネットワークでは、物理的なL2トランクインターフェース間を「透過的なL2ブリッジ」として接続する役割（キャリアネットワークとしての機能）が求められるケースが増えています。しかし、こうしたネットワークには特有の課題があります。複数の顧客を収容する場合、顧客ごとに使用するVLAN IDが重複してしまう可能性があり、さらにインフラ全体を流れるトラフィックが混在してしまう懸念があります。顧客ごとに固有のVLAN IDを割り当てる手法では、顧客側のネットワーク構成が制限されてしまうだけでなく、IEEE 802.1Q規格の限界である「4094個」というVLAN上限をすぐに超えてしまうでしょう。そこで「Q-in-VNI」機能を使うと、サービスプロバイダー側は単一のVLAN（S-VLAN）を用意するだけで、複数のVLAN（C-VLAN）を持つ顧客を収容できるようになります。この方式では、顧客側のVLAN IDはそのまま保持され、たとえ同じS-VLAN内を通っていたとしても、各顧客のトラフィックはネットワーク内で論理的に分離されます。このIEEE 802.1Qトンネリング技術は、タグ付けされたパケットをさらに別のVLANタグで包み込む（VLAN-in-VLAN）という階層構造を利用することで、実質的に利用可能なVLAN空間を拡張します。この機能が設定されたポートは「トンネルポート」と呼ばれ、トンネリング専用のVLAN IDに割り当てられます。サービスプロバイダーは、顧客ごとに固有のS-VLAN IDを割り当てることで、顧客が持つすべてのVLANトラフィックを安全かつ透過的に転送することが可能になります。

## EVPN VXLANファブリックでのQ-in-VNI活用
サービスプロバイダーはQ-in-VNI機能を利用することで、S-VLAN（サービスプロバイダー側のVLAN）をLayer 2 VNIにマッピングし、Layer 2オーバーレイサービスを提供できるようになります。これにより、キャンパス拠点間やデータセンター間をBGP EVPN VXLANで接続し、ビジネス顧客が求めるL2ネットワーク要件に柔軟に応えることが可能になります。

企業ユーザーであっても、EVPN EVI（EVPNインスタンス）を有効にした環境であれば、単一の拠点内でもQ-in-VNIを活用できます。その際、複数のL2セグメントからのトラフィックを特定のS-VLANに集約する形で展開しますが、以下の要件を満たす必要があります。

- ハードウェアの制約: 拠点内のL2VNIオーバーレイセグメント数は、使用するCisco Catalyst 9000シリーズスイッチのサポート上限数に依存します。
- 対称性の確保: ファブリックのエッジ（接続点）全体で、VLANセグメント構成が対称的（左右対称）である必要があります。

S-VLANをEVPNインスタンス（別名：MAC VRF）にマッピングする「Q-in-VNIによるL2オーバーレイサービス」を展開する場合、すべてのC-VLAN（顧客側のVLAN）に含まれるエンドホストのMACアドレス経路（RT2）は、そのS-VLANに対応する単一のブリッジテーブル内で統合的に管理されます。

## BGP EVPN VXLANファブリックでの動作
通常のL2トランク構成の場合、まず入り口側のVTEP（イングレスVTEP）がパケットからIEEE 802.1Qタグを取り除き、VXLANヘッダーを付与してカプセル化し、目的地へ転送します。出口側のVTEP（エグレスVTEP）では、カプセル化を解除（デカプセル化）し、L2VNIをもとのVLANへとマッピングします。エグレス側のポートがトランクポートであれば、対応するVLAN IDがIEEE 802.1Qヘッダーに再び挿入され、ファブリックの外へ送信されます。

Q-in-VNIを設定した場合、VLAN ID 10を持つC-VLAN（顧客側VLAN）からのトラフィックがEVPN VXLANオーバーレイネットワークへ転送されます。このとき、ネットワーク入り口側のQ-in-VNIポートには、プロバイダー側のVLAN 101と、固有のL2 VNI 1001が設定されています。パケットがエッジデバイスのQ-in-VNIトンネルポートに到達すると、VNI 1001を含む外側のVXLANヘッダーで包み込まれます（この際、VLAN 10を含む元の内側のヘッダーはそのまま保持されます）。出口側のエグレスVTEPでは、L2 VNIに基づいて正しいプロバイダーVLAN 101が特定され、そのQ-in-VNIポートへとパケットが転送されます。そして、出口側のトンネルポートから、元のC-VLANタグが付いた状態でパケットが送信されます。

VXLANファブリック内でパケットをキャプチャしてもS-VLANは見えないことに注意してください。VXLANヘッダのVNI値がS-VLANを指し示しています。

## 設定手順

### Step 1

```
enable
```

#### Example
```
Device> enable
```

#### Purpose
Enables privileged EXEC mode. Enter your password, if prompted.

### Step 2
```
configure terminal
```

#### Example:
```
Device# configure terminal
```

#### Purpose
Enters global configuration mode.

### Step 3
```
interface <interface-name>
```

#### Example
```
Device(config)# interface GigabitEthernet1/0/24
```

#### Purpose
Enters interface configuration mode for the interface to be configured as a tunnel port.
This should be the edge port on the VTEP that connects to the interface of the Layer 2 device with a trunk port configuration.

### Step 4
```
switchport access vlan <vlan-id>
```

#### Example
```
Device(config-if)# switchport access vlan 101
```

#### Purpose
Specifies the S-VLAN that is mapped to the L2VNI.

### Step 5
```
switchport mode dot1q-tunnel
```

#### Example
```
Device(config-if)# switchport mode dot1q-tunnel
```

#### Purpose
Sets the interface as an IEEE 802.1Q tunnel port.

### Step 6
```
end
```

#### Example
```
Device(config-if)# end
```

#### Purpose
Returns to privileged EXEC mode.


## 設定例

以下の例では、GigabitEthernet1/0/24が顧客側インタフェースで、このインタフェースを「トンネルポート」として設定します。

トンネルポートに設定するアクセスVLANがS-VLANになり、モードをdot1q-tunnnelにすることでトンネルポートとして動作します。

トンネルポートに `no cdp enable` は自動で設定されます。

```
l2vpn evpn instance {{ evi_number }} vlan-based
 encapsulation vxlan
 replication-type static
!
! S-VLAN mapped to VNI
vlan configuration {{ svlan_number }}
 member evpn-instance {{ evi_number }} vni {{ vni_number }}
!
interface nve1
 no ip address
 source-interface Loopback1
 host-reachability protocol bgp
 member vni {{ vni_number }} mcast-group 225.0.0.101
!
interface GigabitEthernet1/0/24
 ! S-VLAN
 switchport access vlan {{ svlan_number }}
 switchport mode dot1q-tunnel
 no cdp enable
!
```

- **evi_number** EVIの値を格納した変数です
- **svlan_number** S-VLANの値を格納した変数です
- **vni_number** VNIの値を格納した変数です


## 制限事項
IEEE 802.1Q Tunneling のマニュアルに制約事項が記載されています。利用する装置、OSのバージョンにあわせてマニュアルを確認する必要があります。

[Layer 2 Configuration Guide, Cisco IOS XE 17.18.x (Catalyst 9300 Switches)](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/17-18/configuration_guide/lyr2/b_1718_lyr2_9300_cg/configuring_ieee_802_1q_tunneling.html)


### Catalyst 9300 IOS-XE 17.18における制限事項（マニュアルから引用）

IEEE 802.1QトンネリングはLayer 2のパケットスイッチングには適していますが、いくつかのLayer 2機能やLayer 3ルーティングとの間で互換性の制約があります。

- トンネルポートをルーテッドポートにすることはできません。

- IEEE 802.1Qトンネルポートを含むVLANでは、IPルーティングはサポートされていません。トンネルポートから受信されたパケットは、Layer 2情報のみに基づいて転送されます。トンネルポートを含むスイッチ仮想インターフェース（SVI）上でルーティングが有効になっている場合、トンネルポートから受信したタグなしIPパケットは認識され、スイッチによってルーティングされます。顧客はネイティブVLANを通じてインターネットにアクセスできます。このアクセスが不要な場合は、トンネルポートを含むVLAN上にSVIを設定すべきではありません。

- フォールバックブリッジングは、トンネルポートではサポートされていません。トンネルポートから受信されたすべてのIEEE 802.1Qタグ付きパケットは非IPパケットとして扱われるため、トンネルポートが設定されているVLANでフォールバックブリッジングが有効になっていると、IPパケットがVLANをまたいで不適切にブリッジされてしまいます。したがって、トンネルポートを持つVLANでフォールバックブリッジングを有効にしてはなりません。

- トンネルポートは、IPアクセス制御リスト（ACL）をサポートしていません。

- Layer 3の品質サービス（QoS）ACLおよびLayer 3情報に関連するその他のQoS機能は、トンネルポートではサポートされていません。MACベースのQoSは、トンネルポートでサポートされています。

- EtherChannelポートグループ内のIEEE 802.1Q設定が一貫している限り、EtherChannelポートグループはトンネルポートと互換性があります。

- ポート集約プロトコル（PAgP）、リンク集約制御プロトコル（LACP）、および単一方向リンク検出（UDLD）は、IEEE 802.1Qトンネルポートでサポートされています。

- トンネルポートとトランクポートを使用して非対称リンクを手動で設定する必要があるため、ダイナミックトランキングプロトコル（DTP）はIEEE 802.1Qトンネリングと互換性がありません。

- VLANトランキングプロトコル（VTP）は、非対称リンクによって接続されたデバイス間、またはトンネルを介して通信するデバイス間では機能しません。

- ループバック検出は、IEEE 802.1Qトンネルポートでサポートされています。

- ポートがIEEE 802.1Qトンネルポートとして設定されると、スパニングツリーのブリッジプロトコルデータユニット（BPDU）フィルタリングがそのインターフェース上で自動的に有効になります。シスコ検出プロトコル（CDP）は、そのインターフェース上で自動的に無効になります。

- IEEE 802.1Qトンネリングを設定している際、スパニングツリーのBPDUフィルターが自動的に有効になるため、BPDUフィルタリングの設定情報は表示されません。BPDUフィルターの情報は、show spanning tree interface コマンドを使用して確認できます。

- IEEE 802.1QトンネルポートをSPANソースとして設定する場合、パケットロスを回避するために、SVLANに対してスパンフィルターを適用する必要があります。

- IGMP/MLDパケット転送は、IEEE 802.1Qトンネル上で有効にすることができます。これは、サービスプロバイダーネットワーク上でIGMP/MLDスヌーピングを無効にすることで実行可能です。