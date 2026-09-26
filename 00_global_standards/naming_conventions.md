---
domain: global_standards
type: policy
last_updated: 2026-09-21
---

# ネットワーク命名規則 (Naming Conventions)

本ドキュメントは、ネットワーク機器、インターフェース、論理構成要素の命名基準を定義する。
自動化スクリプトやAIによる設定生成を行う際は、必ず本規則に従うこと。

## 1. 機器命名規則 (Device Naming)
機器名は以下のフォーマットに従い、ハイフン区切りで記述する。
`{役割}-{サイト}-{通番}`

- **役割:**
    - `spine`: Spineスイッチ
    - `leaf`: Leafスイッチ
    - `border`: Borderリーフ/ゲートウェイ
- **サイト:** `dc1`, `dc2` 等（3-4文字の略称）
- **通番:** 2桁の数字（例: `01`）

**例:** `leaf-dc1-01`, `spine-dc1-01`

## 2. インターフェース命名規則 (Interface Naming)
物理ポートには必ず物理的な接続先を示す説明（Description）を付与する。

- **書式:** `To-{対向機器名}-{対向ポート}`
- **例:** `To-spine-dc1-01-Ethernet1/1`

## 3. 論理インターフェース命名規則 (Logical Interface Naming)
- **Loopback:**
    - `Loopback0`: Router-ID / BGP Peering用
    - `Loopback1`: VTEP Source Address用
- **VLAN / VRF:**
    - `VRF_TENANT_A`: VRF名（大文字、アンダースコア区切り）
    - `VLAN10`: VLAN名（`VLAN` + ID）

## 4. プロトコル識別子命名規則
- **IS-IS Area:** `49.0001` (全拠点共通)
- **BGP Community:** `65000:VNI`
- **ESI (Ethernet Segment):**
    - `00:00:5e:xx:xx:xx:xx:xx:xx:xx`
    - `xx` 部分は `00-01` (サイトID), `00-01` (ラックID), `01` (ペアID) とすること。

---

## 5. AI/自動化適用時のルール
AIが設定を生成する際、上記にない名前を提案する場合は、以下の順序で候補を算出すること。

1. **既存データの走査:** `03_projects/` 配下の全ファイルをスキャンし、既存の命名と重複がないか確認する。
2. **規則の強制:** 命名規則に準拠していることを確認する。
3. **人間への確認:** ルール外の命名が必要な場合（特例の機器など）は、その理由と共に人間へ承認を求めること。
