# 共通pubspecパッケージ方針

## 目次
1. 広告SDKの標準バージョン
2. 課金SDKの標準バージョン
3. 認証・DB関連の標準バージョン
4. ATT関連パッケージの標準バージョン
5. パッケージ更新の追従方針と現状ログ

---

### 1. 広告SDKの標準バージョン
**背景・意図:** バージョンをアプリごとに固定していないと、Privacy Manifest対応漏れ等の差異が生まれる。以下はMath_Practiceで採用中のバージョンを現時点の標準とする。

**必須要件:**
- `google_mobile_ads: ^5.1.0`を標準バージョンとする（新アプリ立ち上げ時はこのバージョン以上を採用し、更新した場合はここも更新する）

### 2. 課金SDKの標準バージョン
**背景・意図:** 課金は不具合が収益・信頼に直結するため、バージョンの野良運用は避ける。

**必須要件:**
- `in_app_purchase: ^3.2.3`を標準バージョンとする

### 3. 認証・DB関連の標準バージョン
**背景・意図:** Firebase系パッケージはバージョン間の依存関係がシビアで、混在させると不具合の元になる。

**必須要件:**
- 以下をMath_Practiceの実績構成として標準バージョンとする
  - `firebase_core: ^4.7.0`
  - `firebase_auth: ^6.4.0`
  - `cloud_firestore: ^6.3.0`
  - `cloud_functions: ^6.2.0`
  - `firebase_analytics: ^12.4.1`
  - `firebase_crashlytics: ^5.2.2`
  - `google_sign_in: ^7.2.0`
  - `sign_in_with_apple: ^6.1.0`

### 4. ATT関連パッケージの標準バージョン
**背景・意図:** ATT対応はOSアップデートで挙動が変わることがあるため、バージョンを揃えて追従する。

**必須要件:**
- `app_tracking_transparency: ^2.0.7`を標準バージョンとする

### 5. パッケージ更新の追従方針と現状ログ
**背景・意図:** 小規模チーム運用では「最新版に追従すること自体」に価値はなく、追従コスト（破壊的変更の検証工数）の方が大きいことが多い。追従すべきかどうかは「具体的な期限・強制力のある外部要因があるか」「認証・決済等セキュリティ影響が大きい領域か」の2軸で判断し、それ以外は現状維持でよいとする。

**必須要件:**
- 各パッケージの状況確認・判断はこの節に追記していく。確認の都度、確認日を明記して更新すること

**追従要（具体的な理由あり）:**
- `firebase_core` / `firebase_auth` / `cloud_firestore` / `cloud_functions` / `firebase_analytics` / `firebase_crashlytics`（Firebase系全般）: Firebaseが2026年10月にCocoaPodsへの新バージョン配信を停止予定。次の大きな機能追加のタイミング、または同年10月までに、Swift Package Manager移行とあわせてまとめて最新化する（詳細: [flutter_project_structure.md](flutter_project_structure.md)）
- `firebase_analytics` / `GoogleAppMeasurement`: 上記に加え、Privacy Manifest (`PrivacyInfo.xcprivacy`) 未対応が継続中（2026-07-29確認時点で未解決）。監視継続（詳細: [privacy_manifest/sdk_support_status.md](privacy_manifest/sdk_support_status.md)）
- `google_sign_in` / `sign_in_with_apple`: 認証系のため、脆弱性修正が出た場合は追従する（2026-07-29確認時点で既知の緊急対応事項なし）
- `in_app_purchase`: 決済系のため、脆弱性修正が出た場合は追従する（2026-07-29確認時点で既知の緊急対応事項なし）
- Flutter SDK本体: CocoaPods trunkが2026年12月に読み取り専用化予定。SPM移行のためにも3.44系以降へのアップグレードがいずれ必要（2026-07-29時点でMath_Practiceは3.41.6）

**追従不要（現状維持でよい、明確な理由が出るまで追わない）:**
- `google_mobile_ads`: 2026-07-29確認時点で既知の緊急課題なし。SPM対応は済んでいるため、Firebase系移行のタイミングに合わせればよい
- `app_tracking_transparency`: 2026-07-29確認時点で既知の緊急課題なし
- 上記以外のUI/ユーティリティ系パッケージ（`flutter_screenutil` `url_launcher`等）: 明確な追従理由が生じるまで個別追跡しない
