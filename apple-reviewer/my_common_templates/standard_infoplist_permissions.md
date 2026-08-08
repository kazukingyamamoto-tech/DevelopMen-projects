# 標準Info.plist権限説明文

## 目次
1. 使用機能ごとの標準文言
2. SKAdNetworkIdentifier標準リスト
3. NSUserTrackingUsageDescriptionの標準文言

---

### 1. 使用機能ごとの標準文言
**説明:** 権限説明文が曖昧だと審査で理由を問われることがあるため、明確な定型文を用意しておく。

**必須要件:**
- NSCameraUsageDescription等、機能ごとに日本語で理由を明記したテンプレ文を用意する

### 2. SKAdNetworkIdentifier標準リスト
**説明:** 広告ネットワークが増えるたびにIDリストを更新する前提の共通リストとして管理する。

**必須要件:**
- 採用している広告ネットワークのSKAdNetworkIdentifierを常に最新化する（[../advertising/skadnetwork_attribution.md](../advertising/skadnetwork_attribution.md)参照）

### 3. NSUserTrackingUsageDescriptionの標準文言
**説明:** ATTダイアログに表示される説明文で、ユーザーの同意率にも影響する。

**必須要件:**
- 「広告の効果測定とパーソナライズのために使用します」等の定型文を用意する
