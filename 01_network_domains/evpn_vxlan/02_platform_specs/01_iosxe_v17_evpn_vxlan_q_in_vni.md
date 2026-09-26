# Catalyst 9000v IOS-XE 17 固有情報

Cisco Modeling Labs 2.10でEVPN VXLAN Q-in-VNIが動作することを確認しました。

[CMLのラボファイル](./01_iosxe_v17_evpn_vxlan_q_in_vni.yaml)

ルータの設定はCMLのラボファイルに含まれます。

## rapid-PVSTのBPDUについて

実機で確認した結果、VXLANファブリック内は透過しませんでした。BPDUフィルタが効いていると考えられます。
