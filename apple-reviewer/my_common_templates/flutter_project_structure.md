# Flutterプロジェクト共通構成

## 目次
1. ディレクトリ構成の共通テンプレ
2. 状態管理の共通アーキテクチャ選定
3. 環境分岐（dev/prod）の共通方針
4. リリース後に変わりうる公開向けURL・文言の管理方針
5. 真に機密性の高い値（APIキー・秘密鍵等）の管理方針
6. Gitブランチ運用とバージョン管理の共通方針
7. dev/prod起動コマンドの共通化
8. `cloud_firestore`使用時のアーカイブビルド失敗（objective_c.framework）への対処

---

### 1. ディレクトリ構成の共通テンプレ
**背景・意図:** アプリごとに構成がバラバラだと、審査AIやレビュー時に照合コストが増える。UIコードとロジックが同じファイルに混在すると、ロジックの再利用・テスト・DRY原則の徹底が難しくなる。

**必須要件:**
- `lib/ui/`（`screens/` `widgets/`）にはWidget（見た目）のみを置き、通信・計算・永続化等のビジネスロジックを書かない
- 純粋なロジック・データフェッチ・永続化処理は`lib/utils/`に分離する
- ファイル名は短さより「何をしているか」が一目でわかる説明的な命名を優先する（例: `gameRecordLocalAndFirestorePersistenceOperations.dart`）

### 2. 状態管理の共通アーキテクチャ選定
**背景・意図:** アプリ間で状態管理方式が異なると、共通レビュー観点が使いまわせなくなる。Math_Practiceは初学時にRiverpodを知らないまま`StatefulWidget`の`setState`のみで作られ、Firebase Authのサインイン状態や課金状態（is_premium）のチェックが`settingsScreen.dart` `homaPage.dart` `resultScreen.dart`など複数画面に個別に散らばっている。画面数が増えるほど「同じチェックの書き漏れ」が起きやすい構成であり、複数画面で共有する状態を1箇所で管理し変更を検知できるRiverpodのようなライブラリで解決すべき課題だが、完成に近いMath_Practiceを今から移行するのは投資対効果に見合わないため据え置く。

**必須要件:**
- Math_Practiceは`StatefulWidget`/`setState`のまま維持し、無理な移行はしない
- **次に新規で立ち上げるアプリからはRiverpodを標準採用する**
- 上記の切り替えは思いつきで個別判断せず、新アプリ立ち上げ時に本ファイルを起点として適用する

### 3. 環境分岐（dev/prod）の共通方針
**背景・意図:** 開発環境と本番環境の設定を混同すると、テスト用の広告IDやFirebaseプロジェクトが本番に紛れ込む事故が起きる。

**必須要件:**
- flavor（dev/prod）ごとにFirebaseプロジェクト・広告IDを分離する
- 広告ID・プライバシーポリシーURL等の環境依存値は`--dart-define-from-file`で`.env.dev`/`.env.prod`から注入し、コードに直書きしない（背景: 直書きするとdev/prod切り替え忘れの事故や、値未設定時に空文字のまま本番ビルドされるリスクが生じる）
- dev/prodでFirebaseプロジェクト自体を分けていないアプリ（規模的にまだ分ける必要がない場合）では、OAuthクライアントID等Firebaseプロジェクトに紐づく設定値は環境によって変わらないため、dart-define化せず直書き定数にしてよい（背景: 値が環境で変わらないなら`--dart-define-from-file`にする意味がなく、キー渡し忘れで空文字になるリスクだけが残る。OAuthクライアントIDは秘匿情報ではなく`GoogleService-Info.plist`等にも平文で含まれる値のため、直書きしても情報漏洩リスクは増えない）
- `--dart-define`で注入する値（環境依存値・機密ではあるが値そのものは変わらない値のいずれでもないもの）には、dart-define未指定の素のビルドでも安全に動作する`defaultValue`を必ず設定する（背景: 広告ユニットIDのように`defaultValue`未指定の`String.fromEnvironment`は、dart-defineの渡し忘れで空文字になり広告読み込み失敗等の不具合につながる。広告ユニットIDならGoogle公式の公開テスト用ID等、安全に使い回せる値をデフォルトにする）

### 4. リリース後に変わりうる公開向けURL・文言の管理方針
**背景・意図:** プライバシーポリシーURL・利用規約URL・サポートメール・外部記事リンクなどは、Webサイト改修や記事公開タイミングでURLが変わりうる。これらを`String.fromEnvironment`等のコンパイル時定数にすると、値を変更するたびにアプリの再ビルド・再審査提出が必要になる。dev/prod分岐の値（広告ID等、上記3番）と違って環境ごとに値が変わるわけではないため、`--dart-define`化しても更新コストが変わらず恩恵が薄い。一方でFirebase Remote Configには無料枠があり、フェッチ回数は極力抑えたい。またRemote Configは即時反映ではなく「次回起動時＋最小フェッチ間隔経過後」というタイムラグが必ずあるため、ほぼ変わらない値にまで使うと、コスト（フェッチ回数・Console管理・キー名相違による無自覚なフォールバック固定化のリスク）ばかりが増えて恩恵を得られない。したがって「変わりうるものだけをRemote Config化し、それ以外は直書きにする」を徹底する。

**必須要件:**
- 変更確率が十分低い値は、Remote Configではなく**必ず直書き定数にする**。判断基準の目安:
  - 事業判断（契約変更・社名変更等）を経ないと変わらない、または過去に一度も変更されたことがない値
  - 開発者側の裁量が及ばない第三者・プラットフォーム公式の不変URL（例: 既存の`appleReportAProblemUrl` = `https://reportaproblem.apple.com`）
  - Bundle ID・パッケージ名など他の定数から機械的に導出される値
- 上記に該当せず、自社運営のWebサイト改修・記事の移設/非公開化・運用体制変更などで実際に変わりうる公開向けURL・文言（プライバシーポリシー/利用規約/サポート窓口/外部記事リンク等）のみ、コンパイル時定数ではなくFirebase Remote Config等、再ビルドなしで更新できる仕組みで管理する
- dev/prod間で値そのものが変わる環境依存値（広告ユニットID・Firebaseプロジェクト設定等）は引き続き3番の`--dart-define-from-file`方式を使う。両者の使い分け基準は「値が環境で変わるか」ではなく「リリース後に無停止で変えたいか」
- Remote Configにはフェッチ失敗時・未設定時のフォールバック用デフォルト値を必ず設定し、空文字によるリンク切れ表示を防ぐ

### 5. 真に機密性の高い値（APIキー・秘密鍵等）の管理方針
**背景・意図:** 4番のRemote Configも3番の`--dart-define`も、最終的に値がクライアントアプリ（配布されるバイナリ）に渡る点は同じで、リバースエンジニアリングされれば読み取られうる。App Store Server API用の`.p8`秘密鍵のように、漏洩するとApple側に対して任意のAPI呼び出し（購入状態の詐称等）が可能になる真の機密情報は、そもそもクライアントに渡してはならない。Math_Practiceの`functions/src/index.ts`では、この種の値をCloud FunctionsからGoogle Cloud Secret Managerで直接読み込み、クライアントバイナリには一切含めない設計が既に実装されている。

**必須要件:**
- APIの秘密鍵・署名鍵（App Store Server API用`.p8`等）は`firebase functions:secrets:set <NAME>`等でCloud Secret Managerに登録し、Cloud Functions側で`defineSecret('<NAME>')`経由でのみ参照する
- この種の値をFlutterクライアント側（`--dart-define`・Remote Config・ソースコード直書き）へ渡す・埋め込むことは禁止。秘密鍵を要する処理（レシート検証等）は必ずサーバー側で完結させる
- 機密/非機密の判断基準は「漏洩すると第三者になりすまし・不正操作が可能になるか」。迷う場合は機密側に倒す

### 6. Gitブランチ運用とバージョン管理の共通方針
**背景・意図:** dev/prod環境の切り替えを3番（flavor+dart-define）・4番（Remote Config）というビルド時の仕組みに寄せた結果、環境ごとの専用Gitブランチを維持する意味がなくなった。むしろ環境用ブランチは本流との差分が放置されやすく、Math_Practiceでも`prod1.0.0`ブランチが「機能していないため無視してよい」状態で形骸化した実例がある（[[review_logs/Math_Practice/2026-07-26]]参照）。一方でバージョン管理自体は必要で、「どのコミットが実際にどのバージョンとして提出・公開されたか」を後から辿れる手段が要る。

**必須要件:**
- 恒久ブランチは`dev`（作業用トランク。未完成の変更が入っていてよい）と`main`（安定版・出荷可能な状態を指すポインタ）の2本のみとする
- `main`は「リリース確定」時に`dev`の最新コミットへfast-forwardして進める。cherry-pickや個別コミットの選別は行わない（歴史的にcherry-pick運用を規定していたが、実際には一度も使われず常にfast-forward相当で運用されていたため、実態に合わせて簡略化した）
- 一部機能を本番から一時的に隠したい場合は、ブランチ運用（旧`DevOnly`命名規則等）ではなく`kDebugMode`等の実行時フラグ判定でコード側にて制御する
- 環境名を冠した恒久ブランチ（`prod1.0.0`等）は作らない。環境差分は3番・4番のビルド時の仕組みで吸収する
- バージョン管理は`main`上へのタグ付けで行う。`pubspec.yaml`の`version+build`（例: `1.0.0+4`）と一致させたタグ名（例: `v1.0.0+4`）を、App Store Connectへ提出したコミットに付与する
- 例外: 公開済みバージョンに致命的な不具合が見つかり、かつ`dev`が次バージョン向けの未完成の変更で汚れている場合に限り、該当リリースタグ（例: `v1.0.0+4`）から一時的なホットフィックスブランチを切ってよい。修正後は新しいパッチバージョンのタグ（例: `v1.0.1+5`）を打って即リリースし、修正を`dev`にも合流させたうえでホットフィックスブランチは削除する

### 7. dev/prod起動コマンドの共通化
**背景・意図:** flavor機構自体は導入せず（3番参照）、環境差分が広告ユニットIDなど少数の値に限られる場合は、`--flavor`のような大掛かりな仕組みを新設する前に、まず`--dart-define-from-file`＋`defaultValue`の組み合わせだけで足りるかを見極める。Math_Practiceでは環境差分が実質的にiOS広告ユニットID3種のみで、Firebaseプロジェクトもbundle IDも共有のままだったため、flavorを新設せずこの方式で足りると判断した。

**必須要件:**
- 開発時（ローカルでの`flutter run`等）は追加の引数なしで実行してよい。dart-define値には安全なデフォルト値（4番と同じ考え方）をコード側の`defaultValue`で設定しておくことで、これが成立する
- TestFlight配布用のdevビルド（内部テスト用）も同様に、`--dart-define-from-file`を渡さない`flutter build ipa`（引数なし）でよい。`flutter build`系コマンドは常にreleaseモードがデフォルトのため、`kDebugMode`分岐のデバッグ専用機能はこのビルドでも自動的に非表示になる（`flutter run`時のみ表示される）
- 本番リリース・提出候補ビルドには`--dart-define-from-file=.env.prod`を明示的に指定する（`flutter build ipa --dart-define-from-file=.env.prod`）。渡し忘れると安全なデフォルト値（テスト広告等）のまま本番ビルドされてしまうため、リリース手順書・CIスクリプトに明記する
- `.env.dev`のような「デフォルト値と中身が同じだけのファイル」は作らない。dart-define値のデフォルトはコード側の`defaultValue`に一本化し、環境ファイルは実際に本番用の差分がある場合（`.env.prod`）にのみ用意する
- 将来、複数のFirebaseプロジェクトやbundle IDを分ける必要が生じた場合（例: 動作確認用に本番と別アプリとして共存インストールしたい等）は、その時点で改めて`--flavor`導入を検討する。今の設計はそれを妨げない
- 広告ID等の値の環境分岐（dart-define）と、デバッグ専用機能の表示/非表示は別軸で扱う。デバッグ専用機能（デバッグメニュー等）を作る場合、新たな`--dart-define`フラグを増やすのではなく、Flutter標準の`kDebugMode`（`flutter run`で自動的に`true`、リリースビルドで自動的に`false`になる）で分岐させる（背景: Math_Practiceの`settingsScreen.dart`・`firebaseAnalyticsEventLoggingService.dart`で既に実践されている。`--dart-define`を増やすとビルドコマンドの管理項目が増え、渡し忘れの事故リスクも増えるため、ビルドモードで自動的に決まるものはそちらに寄せる）

### 8. `cloud_firestore`使用時のアーカイブビルド失敗（objective_c.framework）への対処
**背景・意図:** `cloud_firestore`はリアルタイムリスナーのためにgRPC-Coreへ依存しており、CocoaPods経由でビルドすると、gRPCのObjective-Cラッパー部分が`objective_c.framework`という独立したフレームワークとして出力される。この際、デバイス向けReleaseアーカイブであるにもかかわらずシミュレータ向けにビルドされた成果物が紛れ込むことがあり、App Store Connectへのアップロード時に「Invalid executable...references an unsupported platform in the [arch] slice」でリジェクトされる（Math_Practiceで実際に発生、[[review_logs/Math_Practice/2026-07-26]]参照）。

**重要な落とし穴（`lipo -info`だけでは不十分）:** Apple Siliconのシミュレータはarm64で動作するため、「arm64だから安全」とは限らない。`lipo -info`はCPUアーキテクチャしか見ておらず、同じ「arm64」でもデバイス向け（`platform IOS`）かシミュレータ向け（`platform IOSSIMULATOR`）かは区別できない。Math_Practiceでは、x86_64スライスをlipoで除去しただけでは解決せず（残ったarm64スライス自体がシミュレータ向けビルドだったため再リジェクトされた）、`vtool -show-build`または`otool -l`で`LC_BUILD_VERSION`の`platform`フィールドを確認して初めて真因が判明した。

**必須要件:**
- 検証には`lipo -info`だけでなく、必ず`vtool -show-build <binary>`（または`otool -l`で`LC_BUILD_VERSION`を確認）を使い、埋め込み済みフレームワークの全スライスが`platform IOS`（`IOSSIMULATOR`ではない）になっていることを確認する
- 根本原因は多くの場合、同一プロジェクトで直前にシミュレータ向けビルド（`flutter build ios --simulator`等）を行った際のビルドキャッシュ（DerivedData/Podsのビルド成果物）が、デバイス向けアーカイブ時に誤って再利用されることにある。**デバイス向けアーカイブビルドの前には、直前にシミュレータビルドを行っていないか確認し、行っていた場合はクリーンビルド（`flutter clean`、必要なら`ios/Pods`と関連DerivedDataの削除、`pod install`のやり直し）を行ってから再アーカイブする**のが確実な対処
- 上記のクリーンビルドに加えて、保険としてRunnerターゲットに「Embed Frameworks」（実体は`[CP] Embed Pods Frameworks`フェーズであることが多い）の後に実行されるRun Scriptビルドフェーズを追加し、万一シミュレータ向けスライスが紛れ込んだ場合に検出・除去できるようにしておく
- この問題は`cloud_firestore`（＝gRPC-Core）に起因するため、Firestoreを使わないアプリでは通常発生しない。Firestoreを新規導入するアプリでは、初回のアーカイブビルド時に同様の検証を行う
