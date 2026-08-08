# 主要SDKのPrivacy Manifest対応状況メモ

## 目次
1. Google Mobile Ads (AdMob) SDKの対応状況
2. Firebase各モジュールの対応状況
3. メディエーション広告ネットワークSDKの対応状況
4. 未対応SDKが見つかった場合の対応方針

---

### 1. Google Mobile Ads (AdMob) SDKの対応状況
**説明:** AdMob SDKは公式にPrivacy Manifestへ対応済みだが、バージョンが古いと反映されない。

**必須要件:**
- 公式が推奨する最新の対応バージョン以上を使用する

### 2. Firebase各モジュールの対応状況
**説明:** Auth/Firestore/Analytics/Crashlytics等モジュールごとに対応時期が異なる場合がある。

**必須要件:**
- 使用している各Firebaseモジュールの対応バージョンを確認する

### 3. メディエーション広告ネットワークSDKの対応状況
**説明:** Unity Ads・AppLovin・Meta Audience Network等、各社の対応状況を個別に追う必要がある。

**必須要件:**
- 採用しているメディエーションSDKごとに対応バージョンを確認する

### 4. 未対応SDKが見つかった場合の対応方針
**説明:** 対応していないSDKを使い続けると審査・アカウント運用上のリスクが残る。

**必須要件:**
- 未対応SDKが見つかった場合は代替SDKへの切り替えを検討する

## 更新ログ
（SDKアップデートの都度、確認日と対応バージョンをここに追記する）

- 2026-07-26: `firebase_analytics` / `GoogleAppMeasurement` 12.13.0 に`PrivacyInfo.xcprivacy`が未同梱であることをMath_Practiceで確認（[../../review_logs/Math_Practice/2026-07-26.md](../../review_logs/Math_Practice/2026-07-26.md)提案8）。他のFirebaseモジュールは対応済み。次回SDK更新時に再確認すること
- 2026-07-29: 本家`firebase-ios-sdk`リポジトリの最新`GoogleAppMeasurement.podspec`（12.18.0時点、Math_Practice未更新）を確認したが、`PrivacyInfo.xcprivacy`への参照は依然なし。FlutterFire側Issueトラッカーでも既知の未解決事項として報告あり。**引き続き未対応、監視継続**
- 2026-07-29追記（関連情報）: Firebaseは2026年10月にCocoaPodsへの新バージョン配信を停止する旨をアナウンス済み（Swift Package Managerへの移行を推奨）。CocoaPods経由でFirebaseを利用しているFlutterプロジェクトは、今後この種のSDK側修正がCocoaPods経由では受け取れなくなる可能性がある点に留意。次回大きな依存関係更新のタイミングでSPM移行の要否を検討する
