---
domain: global_standards
type: policy
category: security
last_updated: 2026-09-21
---

# ネットワークデバイス セキュリティベースライン (Security Baseline)

組織内の全てのネットワークデバイスに対し、適用すべき最低限のセキュリティ要件を定義する。
自動設計やコンフィグ生成の際は、本ベースラインを適用すること。

## 1. 管理・アクセス制御 (Management Plane)
- **SSH アクセス:**
  - SSH Version 2 を必須とする (`ip ssh version 2`)。
  - Telnet は全面禁止とする。
- **タイムアウト:** コンソールおよびVTYセッションは 10分間の非アクティブ状態で自動ログアウトさせる。
- **認証:** 認証はローカルではなく TACACS+ または RADIUS を優先し、ローカル認証はバックアップとしてのみ使用する。

## 2. サービス・ポート制限 (Services & Ports)
- **不要サービスの無効化:**
  - `no ip http server` / `no ip http secure-server` (HTTP関連)
  - `no ip source-route` (ソースルーティングの無効化)
  - `no ip domain lookup` (誤入力によるDNS検索遅延の防止)
  - `no service finger` / `no service pad`
- **CDP/LLDP:** 信頼できないインターフェースでは無効化する。

## 3. ロギングと監視 (Logging & Monitoring)
- **ログの送信:** 全てのログは管理用Syslogサーバに転送する。
- **タイムスタンプ:** ログにはミリ秒単位のタイムスタンプを付与する (`service timestamps log datetime msec`)。
- **SNMP:**
  - コミュニティ名は `public` 等の推測されやすいものを禁止する。
  - SNMPv3 (認証・暗号化あり) を使用する。

## 4. セキュアなパスワード管理
- **暗号化:** パスワードはプレーンテキストでの保存を禁止し、強力な暗号化アルゴリズム（AES-256など）を使用する (`service password-encryption` + パスワードハッシュの強化)。

---

## [AI自動生成時の適用ルール]
AIが設定例を作成する際は、以下のステップを遵守すること。

1. **ベースラインの統合:** 個別の機能設定を作成する前に、このセキュリティベースラインを統合すること。
2. **優先順位:** 機能固有の設定とセキュリティ要件が競合する場合、セキュリティ要件を優先すること。
3. **バリデーション:** コンフィグ生成後、`no ip http server` 等の禁止設定が含まれているか、ベースラインと照合すること。
