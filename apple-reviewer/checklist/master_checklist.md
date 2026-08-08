# マスターチェックリスト

審査前に上から順に確認する。各項目は詳細ファイルにリンク。
広告・課金・返金解約・データ管理の各セクションは「①Apple公式ルール」→「②自分の実装パターン」の2段階で照合する。

## Safety / Performance
- [ ] UGC投稿機能がある場合、通報・ブロック機能があるか → [../apple_guidelines/01_safety/ugc_moderation.md](../apple_guidelines/01_safety/ugc_moderation.md)
- [ ] キッズカテゴリ該当時の追加規制を満たしているか → [../apple_guidelines/01_safety/kids_category.md](../apple_guidelines/01_safety/kids_category.md)
- [ ] プレースホルダーコンテンツ・未完成機能が残っていないか → [../apple_guidelines/02_performance/completeness_crash.md](../apple_guidelines/02_performance/completeness_crash.md)

## Business / Design
- [ ] デジタルコンテンムはApple IAPを使っているか → [../apple_guidelines/03_business/iap_rules.md](../apple_guidelines/03_business/iap_rules.md)
- [ ] サブスクリプションの表示義務事項を満たしているか → [../apple_guidelines/03_business/subscription_rules.md](../apple_guidelines/03_business/subscription_rules.md)
- [ ] 広告関連ビジネスルールに違反していないか → [../apple_guidelines/03_business/advertising_business.md](../apple_guidelines/03_business/advertising_business.md)
- [ ] 最低限の機能性（Webラッパー判定回避）を満たしているか → [../apple_guidelines/04_design/minimum_functionality.md](../apple_guidelines/04_design/minimum_functionality.md)
- [ ] 第三者ログインがある場合、Sign in with Appleを提供しているか → [../apple_guidelines/04_design/sign_in_with_apple.md](../apple_guidelines/04_design/sign_in_with_apple.md)

## Legal / Privacy
- [ ] プライバシーポリシーが最新かつリンク切れがないか → [../apple_guidelines/05_legal/privacy_policy.md](../apple_guidelines/05_legal/privacy_policy.md)
- [ ] アカウント作成機能がある場合、アプリ内削除導線があるか → [../apple_guidelines/05_legal/account_deletion.md](../apple_guidelines/05_legal/account_deletion.md)
- [ ] 第三者データ共有の開示（App Privacy）が実態と一致しているか → [../apple_guidelines/05_legal/data_use_disclosure.md](../apple_guidelines/05_legal/data_use_disclosure.md)
- [ ] ATTプロンプトの実装・文言・IDFA取得タイミングが適切か → [../apple_guidelines/05_legal/att_tracking.md](../apple_guidelines/05_legal/att_tracking.md)
- [ ] 輸出コンプライアンス（暗号化申告）が正しいか → [../apple_guidelines/05_legal/export_compliance.md](../apple_guidelines/05_legal/export_compliance.md)

## Privacy Manifest
- [ ] PrivacyInfo.xcprivacyが必要なSDKすべてに存在するか → [../apple_guidelines/privacy_manifest/overview.md](../apple_guidelines/privacy_manifest/overview.md)
- [ ] Required Reason API使用箇所の申告漏れがないか → [../apple_guidelines/privacy_manifest/required_reason_apis.md](../apple_guidelines/privacy_manifest/required_reason_apis.md)
- [ ] 使用中の主要SDK（AdMob/Firebase等）の対応状況を確認したか → [../apple_guidelines/privacy_manifest/sdk_support_status.md](../apple_guidelines/privacy_manifest/sdk_support_status.md)

## 広告
- [ ] AdMobポリシーに準拠しているか → [../advertising/admob_policy.md](../advertising/admob_policy.md)
- [ ] メディエーション広告ネットワークの規約に準拠しているか → [../advertising/mediation_networks.md](../advertising/mediation_networks.md)
- [ ] 誤タップ誘発・過剰な表示頻度がないか → [../advertising/ad_placement_ux_rules.md](../advertising/ad_placement_ux_rules.md)
- [ ] SKAdNetworkによる広告アトリビューションが設定されているか → [../advertising/skadnetwork_attribution.md](../advertising/skadnetwork_attribution.md)
- [ ] 自分の標準AdMob実装パターンに沿っているか → [../my_common_templates/advertising/admob_implementation_pattern.md](../my_common_templates/advertising/admob_implementation_pattern.md)

## 課金
- [ ] プロダクト設定・価格・ローカライズが正しいか → [../in_app_purchase/product_setup_checklist.md](../in_app_purchase/product_setup_checklist.md)
- [ ] レシート検証をサーバーサイドで行っているか → [../in_app_purchase/receipt_validation_policy.md](../in_app_purchase/receipt_validation_policy.md)
- [ ] 購入の復元(Restore)機能があるか → [../in_app_purchase/restore_purchase_flow.md](../in_app_purchase/restore_purchase_flow.md)
- [ ] Sandboxアカウントでのテストが完了しているか → [../in_app_purchase/sandbox_test_accounts.md](../in_app_purchase/sandbox_test_accounts.md)
- [ ] 自分の標準IAP実装パターンに沿っているか → [../my_common_templates/in_app_purchase/iap_implementation_pattern.md](../my_common_templates/in_app_purchase/iap_implementation_pattern.md)

## 返金・解約
- [ ] 返金導線をApple管理のフローに誘導しているか → [../refund_cancellation/apple_managed_refund_flow.md](../refund_cancellation/apple_managed_refund_flow.md)
- [ ] サブスク解約手順の説明を提供しているか → [../refund_cancellation/subscription_cancel_instructions.md](../refund_cancellation/subscription_cancel_instructions.md)
- [ ] 自分の標準サポート導線パターンに沿っているか → [../my_common_templates/refund_cancellation/support_flow_pattern.md](../my_common_templates/refund_cancellation/support_flow_pattern.md)

## データ管理
- [ ] DB書き込み権限設計が適切か → [../data_management/database_write_policy.md](../data_management/database_write_policy.md)
- [ ] DB削除権限・論理削除/物理削除方針が適切か → [../data_management/database_delete_policy.md](../data_management/database_delete_policy.md)
- [ ] セキュリティルールが共通方針に沿っているか → [../data_management/security_rules_template.md](../data_management/security_rules_template.md)
- [ ] データ保持期間ポリシーを満たしているか → [../data_management/retention_policy.md](../data_management/retention_policy.md)
- [ ] 自分の標準データ設計パターンに沿っているか → [../my_common_templates/data_management/data_storage_pattern.md](../my_common_templates/data_management/data_storage_pattern.md)

## 自分の共通方針（プロジェクト全般）
- [ ] プロジェクト構成が共通テンプレに沿っているか → [../my_common_templates/flutter_project_structure.md](../my_common_templates/flutter_project_structure.md)
- [ ] 共通パッケージ構成から逸脱していないか → [../my_common_templates/common_pubspec_packages.md](../my_common_templates/common_pubspec_packages.md)
- [ ] Info.plist権限説明文が標準テンプレに沿っているか → [../my_common_templates/standard_infoplist_permissions.md](../my_common_templates/standard_infoplist_permissions.md)
- [ ] 審査員向けレビューノート（デモアカウント等）を準備したか → [../my_common_templates/app_review_notes_template.md](../my_common_templates/app_review_notes_template.md)
