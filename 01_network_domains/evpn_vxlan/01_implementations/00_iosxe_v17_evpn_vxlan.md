---
title: Cisco IOS-XE 17.18 限定：Catalyst 9000vにおける Single-Active ESIを用いた EVPN over VXLAN 設定ガイド
topic: EVPN VXLAN / Cisco IOS-XE 17.18
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

# Catalyst 9000vにおける Single-Active ESI を用いたEVPN over VXLAN 設定ガイド

## 1. 概要と適用要件

本ドキュメントは、**Cisco IOS-XE 17.18** が稼働する Catalyst 9000v (C9KV)環境専用のガイドです。
バージョンごとのコマンド構文差異による不具合を防ぐため、対象プラットフォームおよびOSバージョンを厳密に限定して作成されています。

### 1.1 前提環境・17.18における制約事項

#### 1.1.1 対象OS
Cisco IOS-XE 17.18 限定です。他のバージョンではコマンド構文や動作仕様が異なる場合があります。

#### 1.1.2 プラットフォーム
Catalyst 9000v (C9KV) 仮想アプライアンスです。

#### 1.1.3 冗長化モード
ESI `single-active` モードです。IOS-XE 17.18 で動作する C9KV では All-Active 構成は実装されていません。

#### 1.1.4 機能上の制限

IOS-XE 17.18 で動作する C9KV ではESIの `identifier type 3` 設定時に `local-discriminator` パラメータがサポートされません。

IOS-XE 17.18 で動作する C9KV では レプリケーションモードは static しかサポートされていません。マルチキャストルーティングが必要になります。

#### 1.1.5 対向機器接続条件

IOS-XE 17.18 で動作する C9KV ではNon-DF側のLACPサスペンド制御が正しく動作しませんでした。そのため対向スイッチ側ではLACP (Port-Channel)を組まず、**独立した物理ポート接続（STP や Active/Standby チーミング前提）**としてESIを共有する両側の装置に接続する必要があります（Non-DF側の装置にシングル構成で接続してしまうと、通信できません）

---

## 2. パラメータ連携マッピング (Data & Control Plane Mapping)

IOS-XE 17.18 における EVPN over VXLAN のパラメータ連携図です。

```text
[ Access Interface ]  Gi1/0/3 (VLAN 10)
                          │
[ ESI Layer ]         evpn ethernet-segment 10
                          │
[ VLAN/EVI Mapping ]  vlan configuration 10
                          ├── member evpn-instance 10 ──► [ EVPN Control Plane ] l2vpn evpn instance 10 (RD/RT)
                          └── member vni 10010        ──► [ VXLAN Data Plane ] interface nve1 (VTEP)
```

### 2.1 主要識別子一覧
| 識別子 (ID) | 設定値例 | 役割・相互関係 (IOS-XE 17 仕様) |
| :--- | :--- | :--- |
| **VLAN ID** | `10` | アクセスポート (`Gi1/0/3`) と `vlan configuration 10` を紐付け |
| **EVI (EVPN Instance)** | `10` | `vlan configuration 10` と `l2vpn evpn instance 10` (RD/RT定義) を紐付け |
| **VNI (VXLAN ID)** | `10010` | `vlan configuration 10` と `interface nve1` (VXLAN 転送面) を紐付け |
| **ESI (Ethernet Segment)**| `10` | `Gi1/0/3` と `l2vpn evpn ethernet-segment 10` を紐付け |
| **VTEP Source IP** | `Loopback1` | `interface nve1` の VXLAN トンネル送信元 IP アドレス |

---

## 3. 設定例 (Leaf Switch: c9kv-1 / IOS-XE 17.18)

以下は、IOS-XE 17.18 で動作するCatalyst9000vを Single-Active ESI Leaf スイッチにしたときの設定です。

注意: 本設定の適用にあたっては[プラットフォーム固有の制限事項](../02_platform_specs/00_iosxe_v17_evpn_vxlan.md)を必ず確認してください。

```text
!
hostname {{ hostname }}
!
ip routing
ip multicast-routing
!
vtp mode transparent
!
! --- EVPN & ESI グローバル定義 (IOS-XE 17 構文) ---
l2vpn evpn
!
l2vpn evpn ethernet-segment {{ esi_number }}
 identifier type 3 system-mac {{ esi_system_mac }}
 redundancy single-active
!
! --- EVPN インスタンス (制御面: RD/RT) 定義 ---
l2vpn evpn instance {{ evi_number }} vlan-based
 encapsulation vxlan
 rd {{ route_distinguisher }}
 route-target export {{ route_target }}
 route-target import {{ route_target }}
 replication-type static
!
! --- MTU長定義 ---
system mtu {{ system_mtu }}
!
! --- VLAN, EVI, VNI のマッピング定義 (IOS-XE 17 構文) ---
vlan configuration 10
 member evpn-instance {{ evi_number }} vni {{ vni_number }}
!
vlan 10
!
! --- アンダーレイ・VTEP インターフェース定義 ---
interface Loopback0
 description Router-ID/BGP Peering/Management
 ip address {{ loopback0_address }} 255.255.255.255
 ip pim sparse-mode
 ip router isis core
!
interface Loopback1
 description VTEP Source Address
 ip address {{ loopback1_address }} 255.255.255.255
 ip pim sparse-mode
 ip router isis core
!
interface GigabitEthernet1/0/1
 description Uplink to Spine-1
 no switchport
 ip address 192.168.13.1 255.255.255.0
 ip pim sparse-mode
 ip router isis core
 isis network point-to-point
!
interface GigabitEthernet1/0/2
 description Uplink to Spine-2
 no switchport
 ip address 192.168.12.1 255.255.255.0
 ip pim sparse-mode
 ip router isis core
 isis network point-to-point
!
! --- アクセスポート & ESI バインディング ---
interface GigabitEthernet1/0/3
 description Downlink to Access Switch / Host
 switchport trunk allowed vlan 10
 switchport mode trunk
 evpn ethernet-segment {{ esi_number }}
!
! --- VXLAN NVE インターフェース定義 ---
interface nve1
 no ip address
 source-interface Loopback1
 host-reachability protocol bgp
 member vni {{ vni_number }} mcast-group 239.1.1.10
!
! --- アンダーレイルーティング (IS-IS) ---
router isis core
 net 49.{{ isis_area_number }}.{{ isis_system_id }}.00
 is-type level-2-only
 router-id Loopback0
 metric-style wide
 log-adjacency-changes
!
! --- オーバーレイ制御面 (BGP EVPN) ---
router bgp 65000
 bgp router-id interface Loopback0
 bgp log-neighbor-changes
 {% for neighbor in bgp_neighbors %}
 neighbor {{ neighbor }} remote-as 65000
 neighbor {{ neighbor }} update-source Loopback0
 {% endfor %}
 !
 address-family l2vpn evpn
  {% for neighbor in bgp_neighbors %}
  neighbor {{ neighbor }} activate
  neighbor {{ neighbor }} send-community extended
  {% endfor %}
 exit-address-family
!
ip pim rp-address {{ rp_address }}
!


```

* **hostname** ルータのホスト名の変数です
* **system_mtu** システムのMTU長の変数です
* **isis_area_number** 2バイトのISISエリア番号を格納する変数です（例：0001）
* **isis_system_id** 6バイトのシステムIDを4オクテットごとにドットで区切ったものを格納する変数です（例：0000.0000.0001）
* **loopback0_address** ルータのLoopback0のアドレスを格納する変数で、マスク指定は含みません（例：192.168.255.1）
* **loopback1_address** ルータのLoopback1のアドレスを格納する変数で、マスク指定は含みません（例：192.168.254.1）
* **vni_number** VXLANのVNI番号の変数です
* **evi_number** EVPNのEVI番号の変数です
* **esi_number** 同じLANセグメントを共有するESI番号の変数です
* **esi_system_mac** 同じESIで共有するMACアドレスの変数です
* **route_distinguisher** Route Distinguisher値の変数です
* **route_target** Route Target値の変数です
* **bgp_neighbors** BGPネイバーのリストを格納する変数です
* **rp_address** マルチキャストルーティングのランデブーポイントのIPアドレスを格納する変数です

---

## 4. コンポーネント別構成解説

### 4.1 `vlan configuration` と `evpn-instance` の紐付け理由
IOS-XE 17.18 における `vlan configuration 10` 配下の `member evpn-instance 10
vni 10010` 設定は、データ面（VNI）と制御面（EVI）を直接統合します。
* **VNI 10010:** VLAN 10 のフレームを VXLAN カプセル化するための識別子（データ面）
* **evpn-instance 10:** 該当 VLAN の MAC/IP 学習情報をどの BGP
Route-Distinguisher (RD) および Route-Target (RT) で広報するかを決定するポリシールール（制御面）

### 4.2 Single-Active ESI の挙動
* **DF 選出 (Designated Forwarder Election):** 同一の `system-mac`
(`0000.0000.0001`) を持つ Leaf 間で BGP Type 4 ルートを交換し、どちらの Leaf
がトラフィックを転送するか（DF）を決定します。
* **フォワーディング:** DF に選出された Leaf のみがアクティブとしてトラフィックを転送し、Non-DF 側の Leaf
は該当 ESI ポートでの転送をブロック（ドロップ）します。

---

## 5. 運用確認・トラブルシューティングガイド (IOS-XE 17.18 コマンド)

### 5.1 EVPN Route-Type 1 と Route-Type 4 の役割比較

マルチホーミング制御において、Route-Type 1 と Route-Type 4 は目的が明確に分かれています。

| 項目 | Route-Type 4 (Ethernet Segment Route) | Route-Type 1 (Ethernet
Auto-Discovery Route) |
| :--- | :--- | :--- |
| **主な目的** | 同一 ESI を共有する Leaf スイッチの**発見と DF 選出** |
**エイリアシング・高速コンバージェンス・マルチホーミング制御** |
| **広報単位** | **ES（Ethernet Segment）単位** | **EVI 単位** または **ES 単位** |
| **主な役割** | ① 同一 ESI 設定の対向 Leaf の自動発見<br>② 単一 Leaf のみが転送する **DF 選出**
の実行 | ① **Aliasing (分散転送)**: 未学習 MAC でもマルチホーミングパスへ分散<br>② **Mass
Withdraw (高速迂回)**: 障害時に一括でルートを無効化 |
| **送信契機** | ESI インターフェースの UP 時 | EVI / ESI 有効化時 |

### 5.2 ESI / DF 状態の確認
IOS-XE 17.18 で Ethernet Segment の認識状況および DF / Non-DF 選出結果を確認します。
```bash
show l2vpn evpn ethernet-segment
```
* **確認ポイント:** 両 Leaf で同一 ESI が認識されているか、片方が `DF`、もう片方が `Non-DF` に選出されているかを確認します。

### 5.3 BGP EVPN ルート情報の確認 (IOS-XE 17.18 コマンド)

#### Route-Type 4（DF選出状況の確認）
```bash
show bgp l2vpn evpn route-type 4
```
* **役割:** 同一 ESI を広報している対向 Leaf（Originating Router）を正常に認識できているか確認します。

#### Route-Type 1（自動発見・障害制御ルートの確認）
```bash
show bgp l2vpn evpn route-type 1
```
* **役割:** EVI 単位 / ES 単位の Auto-Discovery ルートが交換され、障害時の Mass Withdraw
や冗長制御が正常に稼働可能か確認します。

### 5.4 VXLAN トンネル・VNI 状態確認
```bash
# IOS-XE 17.18 における NVE インターフェースおよび VNI マッピング動作確認
show nve vni
```