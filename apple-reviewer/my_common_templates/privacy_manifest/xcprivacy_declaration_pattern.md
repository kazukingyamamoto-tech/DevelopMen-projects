# Privacy Manifest (PrivacyInfo.xcprivacy) 実装の自分パターン

Apple公式ルールではなく、複数アプリで統一する自分のPrivacy Manifest実装方針を記録する場所。
Apple公式ルールとの整合性は [../../apple_guidelines/privacy_manifest/](../../apple_guidelines/privacy_manifest/) 配下を参照。

## 目次
1. アプリ本体用PrivacyInfo.xcprivacyの配置と組み込み
2. NSPrivacyTrackingの判定基準
3. NSPrivacyTrackingDomainsの判定基準
4. NSPrivacyCollectedDataTypesの申告方針
5. NSPrivacyAccessedAPITypesの省略可否判断

---

### 1. アプリ本体用PrivacyInfo.xcprivacyの配置と組み込み
**背景・意図:** サードパーティSDK側のPrivacyInfo.xcprivacyは各Podに同梱されるが、アプリ本体用は自分で用意しない限り存在せず見落としやすい（Math_Practiceで実際に欠落していた実例。[[review_logs/Math_Practice/2026-07-26]]参照）。Flutterプロジェクトでは`ios/Runner/`配下に置く必要がある。

**必須要件:**
- `ios/Runner/PrivacyInfo.xcprivacy`を新規作成し、Runnerターゲットの`project.pbxproj`（PBXFileReference / PBXBuildFile / Resourcesビルドフェーズ / トップレベルgroup）に登録する
- ビルド後に生成される`Runner.app`直下に本ファイルが実際にコピーされることを確認する
- `plutil -lint`で生成したplistの構文を確認する

### 2. NSPrivacyTrackingの判定基準
**背景・意図:** ATT許可結果に応じてパーソナライズ広告の有無を切り替えている場合、それはApple社のtracking定義に該当する。広告SDK（Google Mobile Ads等）自身がDeviceID等の項目でTracking=trueと申告していることも判断材料になる。

**必須要件:**
- ATTの許可結果でパーソナライズ広告の出し分けを行っているアプリは`NSPrivacyTracking=true`とする
- 使用中の広告SDKのPrivacyInfo.xcprivacyでTracking=trueの項目がある場合、それも判断根拠に含める

### 3. NSPrivacyTrackingDomainsの判定基準
**背景・意図:** アプリ自身のコードが直接トラッキング用ドメインに通信せず、全て広告SDK経由である場合、SDK側のPrivacyInfo.xcprivacyで既にドメインが申告されているため、アプリ側で重複させる必要はない。

**必須要件:**
- アプリ自身のコードから直接トラッキングドメインへ通信していない場合は空配列のままでよい
- 独自にトラッキングドメインへ通信するコードがある場合のみ追加する

### 4. NSPrivacyCollectedDataTypesの申告方針
**背景・意図:** Firestoreに保存するデータはアプリ本体が収集主体であり、SDK側のマニフェストではカバーされない。Firebase Authのuid・メールアドレス・アプリ固有のユーザーコンテンツ（学習記録等）は典型的な申告対象になる。

**必須要件:**
- Firebase Auth uidは`User ID`として申告する
- メールアドレスは`Email Address`として申告する
- Firestoreに保存するアプリ固有のユーザー生成データ（学習記録等）は`Other User Content`として申告する
- 上記3種は通常`Linked=true`、`Tracking=false`、`Purpose=App Functionality`とする（広告トラッキングに使っていない限り）

### 5. NSPrivacyAccessedAPITypesの省略可否判断
**背景・意図:** Required Reason APIの使用申告は、使用中の全プラグイン/SDKが自身のPrivacyInfo.xcprivacyで既に同梱・申告済みであれば、アプリ本体側での重複申告は不要。

**必須要件:**
- Runner自身のコードで直接Required Reason API該当のAPIを使用していないか確認する
- 使用中の全プラグイン/SDKが各自PrivacyInfo.xcprivacyを同梱済みであることを確認できれば、アプリ本体側では省略してよい
- 同梱していないプラグインがある場合は、そのAPIの使用有無をXcodeのビルド警告で確認する
