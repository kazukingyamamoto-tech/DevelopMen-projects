# SKAdNetwork 広告アトリビューション

## 目次
1. Info.plistへのSKAdNetworkIdentifier登録要否
2. 各広告ネットワークIDの管理
3. ATT拒否時の計測方針

---

### 1. Info.plistへのSKAdNetworkIdentifier登録要否
**説明:** 広告経由のインストール計測を行う場合、ATTの同意有無に関わらず使える集計の仕組みとしてSKAdNetworkがある。

**必須要件:**
- 広告ネットワーク経由のインストール計測を行う場合、Info.plistのSKAdNetworkItemsに識別子を登録する

### 2. 各広告ネットワークIDの管理
**説明:** ネットワークごとに公開しているIDリストが更新されることがある。

**必須要件:**
- 採用している各広告ネットワークが公開する最新のID一覧を定期的に取り込む

### 3. ATT拒否時の計測方針
**説明:** SKAdNetworkはATT拒否時でも集計可能な仕組みであることを理解し活用する。

**必須要件:**
- ATT拒否時のアトリビューション計測はSKAdNetworkに委ねる方針を明確にする
