# projectTemplate

Flutterネイティブアプリ（iOS/Android）を新規開始する際の雛形フォルダです。
`Math_Practice` プロジェクトで運用しているAI開発ルール（`CLAUDE.md`・テスト仕様書運用・`utils`インデックス管理）を、アプリ固有の内容を除いて抽出しています。

## 使い方

1. `flutter create <new_project_name>` で新規Flutterプロジェクトを作成する。
2. このテンプレート内の以下のファイル・フォルダを、作成したプロジェクトのルートにコピーする。
   - `CLAUDE.md`
   - `.env.example`（コピー後、`.env.prod` にリネームして本番用の実際の値を入力する。`.env.prod` はコミットしないこと）
   - `docs/test_specs/`（`README.md` と `_TEMPLATE.md`）
   - `docs/PENDING_DECISIONS.md`
   - `lib/utils/_AI_INDEX.md`
   - `.gitignore` の内容（既存の `flutter create` 生成分とマージし、`.env`/`.env.prod` 除外ルールを追加する）
3. `lib/ui/screens/`・`lib/ui/widgets/`・`lib/utils/` のディレクトリ構成を維持し、CLAUDE.mdのルール（UIとロジックの分離、長文命名規則、DRY原則）に沿って実装を進める。
4. 環境変数の読み込みは `flutter_dotenv` などの外部パッケージを使わず、`String.fromEnvironment('変数名', defaultValue: '...')` を使用する。開発時は安全なデフォルト値へのフォールバックにより素の `flutter run` でよく、本番ビルド時のみ `--dart-define-from-file=.env.prod` を指定する（CLAUDE.md 2章参照）。
5. Gitブランチ運用は `dev` を作業ブランチ、`main` を安定版とする2ブランチ運用を基本とする。`main` への直接コミットは禁止。`dev` はコミットのたびに `git push origin dev` まで行う。`main`は`dev`の履歴上のある地点をそのまま指すポインタとして扱い、リリース確定時に`dev`の最新コミットへfast-forwardした上で注釈付きタグ（例: `v1.0.0+4`）を付与する（CLAUDE.md 5章・7章参照）。一部機能を本番から一時的に隠したい場合はブランチ運用ではなく`kDebugMode`等の実行時フラグで制御する。
6. ユーザーが対応を保留した事項は `docs/PENDING_DECISIONS.md` に記録し、解決したら該当エントリを削除する（CLAUDE.md 8章参照）。
7. このプロジェクトが `apple-reviewer` フォルダと同じ階層（`flutterprojects` 直下）に置かれている場合、審査ナレッジベースとの連携ルール（CLAUDE.md 9章）が有効になる。将来App Storeへ提出する予定であれば `../apple-reviewer/apps/{アプリ名}.md` の新規作成を検討する。

## フォルダ構成

```
projectTemplate/
├── CLAUDE.md                  # AIエージェント向け開発ルール（DRY・環境変数・テスト駆動・審査対策・Git運用・SOP・main/dev運用・保留管理・審査連携）
├── README.md                  # このファイル
├── .env.example                # 環境変数のサンプル（実値は.env.prodに記入しコミットしない）
├── .gitignore                  # Flutter標準 + .env/.env.prod除外ルール
├── docs/
│   ├── PENDING_DECISIONS.md   # 保留事項の記録先（iOS/Android/共通で区分）
│   └── test_specs/
│       ├── README.md          # テスト仕様書フォルダの運用ルール
│       └── _TEMPLATE.md       # 新規テスト仕様書作成時に必ずコピーするフォーマット
└── lib/
    ├── ui/
    │   ├── screens/            # 画面（Widget）。ビジネスロジックを直接書かない
    │   └── widgets/            # 再利用可能な部品Widget
    └── utils/
        └── _AI_INDEX.md        # 純粋関数（ロジック）のインデックス。新機能実装前に必ず確認・更新する
```

## このテンプレートの前提とするルールの要点

- **DRY原則**: 新機能実装前に `lib/utils/_AI_INDEX.md` を確認し、重複実装を避ける。
- **命名規則**: `lib/utils/` 配下の関数・ファイル名は説明的な長文キャメルケースにする。
- **環境変数**: APIのURLやクライアントIDなどはハードコードせず `.env.prod` + `--dart-define-from-file` で管理する。コード側は `defaultValue` で安全な値へフォールバックさせ、開発時はdart-defineファイル無しで動かせるようにする。
- **テスト駆動**: 実装前に `docs/test_specs/_TEMPLATE.md` に沿った仕様書を作成し、人間の承認を得てから着手する。実装完了後はAI自身で検証可能な項目（`flutter analyze`・ビルド・疎通確認等）を実際に検証してチェックする。
- **審査対策**: カメラ・写真・位置情報等にアクセスする機能は同意フローを先に組み込む。
- **Git運用**: `dev` ブランチで作業し、実装前に未コミット差分があれば自動要約してコミットしてからクリーンな状態で着手する。コミット後は毎回 `git push origin dev` まで行う。`main` への直接コミット・`git merge` による同期は禁止。
- **一部機能を本番から隠す場合**: ブランチ運用ではなく `kDebugMode` 等の実行時フラグで制御する。
- **mainへのリリース確定**: `main` は `dev` の履歴上のある地点をそのまま指すポインタとして扱い、リリース確定時に `dev` の最新コミットへfast-forwardする（cherry-pickや個別コミットの選別は行わない）。
- **リリースタグ**: `main` へのリリース確定完了時は、バージョン管理ブランチではなく注釈付きGitタグ（`pubspec.yaml`の`version+build`と一致させた`v1.0.0+4`等）を付与する。
- **保留事項の管理**: 対応を保留した事項は `docs/PENDING_DECISIONS.md` に理由・現状・再開条件とともに記録し、解決したら削除する。
- **審査ナレッジベース連携**: `apple-reviewer` と同じ階層に置かれている場合、`../apple-reviewer/review_logs/{アプリ名}/` を通じたレビュー指摘の受け渡しルールが適用される。
