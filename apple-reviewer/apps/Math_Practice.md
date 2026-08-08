# アプリプロファイル: Math_Practice（100MATH）

review_logsやチェックリストの照合対象範囲を決めるための、このアプリの現状スナップショット。
実装が変わったら都度このファイルを更新する（過去の履歴は`../review_logs/Math_Practice/`側で管理し、ここは常に最新状態のみを反映する）。

## 基本情報
- プロジェクトパス: `../../Math_Practice`（このリポジトリと兄弟フォルダ）
- 表示名: 100MATH / パッケージ名: math_practice
- Bundle ID: `com.zetaiw.math100`
- カテゴリ: 算数計算練習ゲーム（学年別モード）。App Store Connect上でのKidsカテゴリ指定有無は審査時に要確認
- バージョン: 1.0.0+4（pubspec.yaml時点）

## Gitブランチ運用
- `dev` — 現在最終調整中の最新開発ブランチ。**レビューは基本的にこのブランチを対象にする**
- `main` — dev確定後にマージされるリリース先。まだdevの内容は未反映
- `prod1.0.0` — 機能していないため無視してよい
- 上記以外のリモートブランチ（feature/*, chore/*等）は作業用のため通常は無視

## 機能フラグ（チェックリストのスコープ判定用）
- UGC（投稿・SNS的機能）: **なし** → `apple_guidelines/01_safety/ugc_moderation.md`系のチェックはスキップ可
- 第三者ログイン: **あり**（Google Sign-In）→ Sign in with Apple対応も実装済み（`sign_in_with_apple`パッケージ使用）
- 匿名アカウント運用: あり（匿名購入からのアカウント引き継ぎ・データマージ機能: `anonymousPurchaseAccountLinkingService.dart` `data_merge_screen.dart`）
- 広告（AdMob）: **あり**（バナー/インタースティシャル/リワード、`adMobBannerInterstitialAndRewardedAdManager.dart`）。ATT実装済み（`NSUserTrackingUsageDescription`設定済み・consentScreen.dartあり）、SKAdNetwork設定済み
- 課金（IAP）: **あり**。ただしサブスクリプションではなく非消耗型「広告削除」1商品のみ（`removeAdsProductId = com.zetaiw.math100.removeads`）→ サブスク解約導線系のチェック項目は対象外、Apple管理の返金導線チェックのみ対象。購入の逆引き（`purchase_index`）は「現在の所有uidを1つだけ持つ」Transfer方式（2026-08-08に採用、`my_common_templates/in_app_purchase/iap_implementation_pattern.md`4番参照）。`restorePurchases()`の自動呼び出しは既存アカウントへの切り替えフォールバック時（`credential-already-in-use`等）のみに限定されており、新規Firebaseアカウント作成時（`loginScreen.dart`の`_handlePostSignIn`）には自動実行しない（2026-08-08に廃止）。設定画面の「購入を復元」ボタンによる手動復元は認証状態を問わず常時利用可能
- データ保存: Firestore使用（学習記録の保存・ローカルからのマイグレーション処理あり: `gameRecordLocalAndFirestorePersistenceOperations.dart` `localRecordsToFirestoreMigrationService.dart`）
- アカウント削除導線: あり（`settingsScreen.dart`に実装。再認証 → Cloud FunctionsでのFirestore再帰削除 → Firebase Auth削除。匿名ユーザーには非表示）
- 公開向けURL・文言の管理: プラポリURL/利用規約URL/サポートメール/「解き方を学ぶ」リンクの4値はFirebase Remote Config管理（Firebaseプロジェクト`math-a9588`、`lib/utils/remoteConfigUrlEmailValuesFetchService.dart`。フォールバック値はコード内に保持）。Firebase Console側のパラメータ登録・公開は完了済み（2026-07-29）

## ビルドコマンド
- flavor機構は未導入（`my_common_templates/flutter_project_structure.md`3番参照。環境差分がiOS広告ユニットID3種のみのため、現時点ではflavor不要と判断）
- 開発時（ローカル実行）: 追加引数なしの素の`flutter run`（広告ユニットIDは`String.fromEnvironment`の`defaultValue`によりGoogle公式の公開テスト広告IDへ自動フォールバック）
- TestFlight配布用devビルド（内部テスト用）: 追加引数なしの`flutter build ipa`（開発時と同じくテスト広告ID。releaseモードのため`kDebugMode`分岐のデバッグ専用機能は非表示）
- 本番リリース・提出候補ビルド: `flutter build ipa --dart-define-from-file=.env.prod`（`.env.prod`はGit管理外、ローカルにのみ存在）
- `.env.dev`は2026-07-29に削除済み（`defaultValue`と内容が重複し冗長だったため）

## 主要パッケージ（pubspec.yaml抜粋）
- 広告: `google_mobile_ads: ^5.1.0`
- 課金: `in_app_purchase: ^3.2.3`
- 認証/DB: `firebase_core ^4.7.0` / `firebase_auth ^6.4.0` / `cloud_firestore ^6.3.0` / `google_sign_in ^7.2.0` / `sign_in_with_apple ^6.1.0` / `cloud_functions ^6.2.0`
- ATT: `app_tracking_transparency: ^2.0.7`
- 分析/クラッシュ: `firebase_analytics ^12.4.1` / `firebase_crashlytics ^5.2.2`

## 適用中のmy_common_templatesパターン
- 本アプリのレビューを通じて`my_common_templates/`の大半のパターンが整備された（広告/Privacy Manifest/データ管理/返金導線/プロジェクト構成の各ドメイン）。詳細は各ファイルと[[review_logs/Math_Practice/2026-07-26]]を参照
