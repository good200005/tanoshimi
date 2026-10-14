# タノシミ (tanoshimi / 타노시미)

[한국어](README.md) | **日本語**

> 日韓交流旅行のパーティーマッチング + リアルタイム共同プランニング + AI旅行おすすめサービス

**「楽しみ(タノシミ)」= 期待・ワクワク。** 旅行に*一緒に行く人探し*から*プラン作成*、*帰ってからの記録*までを、ひとつのサービスでつなぎます。

## デモ

| | リンク | 説明 |
|---|---|---|
| **本番サービス** | https://tanoshimi.onrender.com | ログイン・パーティー・プランナー・チャットなど全機能が動作 (Render + TiDB Cloud) |
| **静的プレビュー** | https://good200005.github.io/tanoshimi/ | サーバーなしで画面のみ確認 (GitHub Pages、機能は動作しません) |

- 無料プランのため、しばらくアクセスがないとスリープし、**初回アクセスは30〜60秒ほどかかります**。
- デモアカウント: `yuja@test.com` / `Test1234!`
- アップロード画像はサーバー再起動時に初期化されます。

---

## 解決する課題

| 課題 | 解決策 |
|---|---|
| 日韓旅行で、**条件(性別・年齢・国籍・旅行スタイル)に合う同行者**を探せるチャネルがない | 条件ベースのパーティーマッチング |
| 複数人でのプラン作成で、**LINE + Excel + 地図**を行き来し、最新版が管理できない | 編集権(ロック)方式のリアルタイム共同編集 + スナップショットによるロールバック |
| アクティビティ選びで、**天気・屋内外・動線**を一つずつ確認する必要がある | AIによるアクティビティ推薦・動線ソート・実現性チェック |

## 主な機能

| モジュール | 内容 |
|---|---|
| **認証・会員** | メール認証による登録、ログイン、ログイン失敗ロック、ソーシャルログイン (Google・Naver) |
| **パーティー(即席募集)** | 開設・募集・申請・承認・退場・完了、性別/年齢/国籍による参加条件 |
| **共同プランナー** | パーティーごとに計画表1つ、分単位のドラッグ編集、編集権ロック、自動/手動保存、スナップショット、賛否投票 |
| **AI推薦・チャットボット** | 同行チャットボット、屋内外(venue_type)のAI判定 + キャッシュ、天気に基づく再推薦、チャット翻訳、AIクレジットの1日上限 |
| **AI動線最適化** | 固定(航空・宿泊)/移動可能(アクティビティ)を区別し、座標ベースの最近傍法で並べ替え |
| **コミュニティ** | 旅行掲示板(投稿・コメント・いいね)、おすすめ投稿、フォロー、パーティーチャット (WebSocket)、1:1 DM、ユーザーブロック |
| **マイページ** | プロフィール、参加中パーティー・完了した旅行、地域制覇ヒートマップ、称号(38種/8カテゴリ)、マナー温度、プロフィール背景のカスタマイズ |
| **管理者** | 会員の停止/復旧、パーティー強制終了、通報処理、バナー管理、管理者権限管理 |
| **通知・サポート** | アプリ内通知 (フォロー・コメント・パーティー申請/承認/拒否/退場)、1:1問い合わせ掲示板 |

## 技術スタック

| レイヤー | 技術 |
|---|---|
| 言語/ランタイム | Java 21 |
| フレームワーク | Spring Boot 3.5.15 (Web MVC · Data JPA · Security · OAuth2 Client · WebSocket · WebFlux WebClient) |
| ビルド | Gradle (Groovy DSL) |
| ビュー | Thymeleaf + `thymeleaf-extras-springsecurity6` (サーバーサイドレンダリング) |
| フロントエンド | Vanilla JS (`static/js/*.js`)、単一の `app.css`、SockJS + STOMP — Nodeビルドなし |
| DB | MySQL 8 (`utf8mb4`)、`schema.sql` を単一の原本とし、番号付きマイグレーション、Hibernate `ddl-auto: update` を補助的に使用 |
| リアルタイム | STOMP over WebSocket — エンドポイント `/ws`、ブローカー `/topic`、アプリプレフィックス `/app` |
| ファイル/画像 | Apache Tika (形式検証)、metadata-extractor (EXIF GPS)、webp-imageio、AWS S3 / Rekognition SDK |
| 外部連携 | SMS (Aligo) · 天気 (KMA/OpenWeather) · 翻訳 (Papago/DeepL) · AI (Anthropic/Gemini) — **すべてデフォルトは `mock`**、キーは `application-local.yml` に設定 |
| テスト | JUnit 5、`spring-security-test`、H2 (テスト実行時) |

### 設計上のポイント

- **外部連携はすべて `mock` がデフォルト**: APIキーがなくても開発・デモが可能。本番では設定の切り替えのみで実サービスに接続します。
- **機密情報の分離**: DB認証情報・OAuth secret・外部APIキーは、Git管理外の `application-local.yml` または環境変数でのみ注入します。
- **日韓2言語対応**: `messages_ko.properties` / `messages_ja.properties` による画面の多言語化。
- **定期バッチ**: 不要ファイルの削除 (毎日04:00)、パーティー完了処理 (毎日00:30)。

## クイックスタート

> ソースコードはリポジトリの [`tanoshimi/`](tanoshimi/) ディレクトリにあります。以下のコマンドはその中で実行してください。

### 1. DBの準備 (MySQL 8)

```bash
cd tanoshimi
mysql -u root -p < src/main/resources/db/schema.sql
mysql -u root -p tanoshimi < src/main/resources/db/data.sql   # デモデータ (全アカウントのパスワード: Test1234!)
# 続けて db/ フォルダの migration_v16_* 〜 migration_v20_* をファイル名順に実行
```

> **正確なマイグレーション順序とデモデータ(ヒートマップ・称号)の設定は [`tanoshimi/src/main/resources/db/README.md`](tanoshimi/src/main/resources/db/README.md) に従ってください。** アプリは `ddl-auto: update` で起動するため、`schema.sql` 実行後にサーバーを一度起動して停止すると、不足カラムが自動補完されます。

### 2. 個人用シークレットファイルの作成 (Git管理外)

```bash
cp src/main/resources/application-local.yml.example src/main/resources/application-local.yml
# -> DBアカウント、(任意) OAuth / 外部APIキーを入力
```

- 外部連携はすべて `mock` がデフォルトのため、**キーなしでも開発・デモが可能**です。
- メール認証のローカル既定値は `provider: log` で、認証コードが**アプリのコンソールログ**に出力されます。
- ソーシャルログインは `application.yml` に Google・Naver・LINE の3つが登録されているため、`application-local.yml` で3つすべてに `client-id`/`client-secret` を設定しないとアプリが起動しません (未使用のプロバイダーはダミー値で可)。

### 3. 起動

```bash
./gradlew bootRun     # http://localhost:8080  (プロファイル: local)
```

### テスト

```bash
./gradlew test
```

## プロジェクト構成

```
tanoshimi/                      # リポジトリルート
├── README.md                   # 韓国語版
├── README.ja.md                # (このファイル)
├── docs/                       # 開発成果物ドキュメント
└── tanoshimi/                  # Spring Boot アプリケーション
    ├── build.gradle
    └── src/
        ├── main/
        │   ├── java/net/datasa/tanoshimi/
        │   │   ├── controller/         # View + @RestController (27個)
        │   │   ├── service/            # ドメインサービス
        │   │   ├── repository/         # Spring Data JPA
        │   │   ├── domain/{entity,dto} # エンティティ / DTO
        │   │   ├── scheduler/          # FileCleanup(04:00), PartyCompletion(00:30)
        │   │   ├── config/ · auth/ · exception/
        │   └── resources/
        │       ├── templates/          # Thymeleaf
        │       ├── static/{css,js,vendor}
        │       ├── db/                  # schema.sql + migration_v16~ + data.sql
        │       ├── application.yml
        │       └── messages_ko/ja.properties
        └── test/
```

## ドキュメント

> 現在の設計書は韓国語です。コードとドキュメントが異なる場合は**コードが正**です。

| ドキュメント | 内容 |
|---|---|
| [01. プロジェクト企画書](docs/01-프로젝트-기획서.md) | サービス概要、目標、対象ユーザー、主要機能、技術スタック、チーム役割、スケジュール |
| [02. 要件定義書](docs/02-요구사항-정의서.md) | モジュール別の機能要件(FR)、非機能要件(NFR)、エラーコード |
| [03. 画面設計書 (UI)](docs/03-화면설계서.md) | 画面一覧、ルーティング、画面別の構成要素・動作、フロントエンド構成 |
| [04. ERD・テーブル設計書](docs/04-ERD-테이블설계서.md) | エンティティ関連図、テーブル定義、主要カラム・制約・インデックス |
| [db/ フォルダ案内](tanoshimi/src/main/resources/db/README.md) | ローカルDBのセットアップ順序、マイグレーション・シードファイルの説明 |

## チーム

| 役割 | 担当 |
|---|---|
| チームリーダー / 統合・QA | アーキテクチャ、ブランチ戦略、統合・QA、AI動線最適化、マナー温度、通報/管理者の一部 |
| 認証 | 会員登録・ログイン・ソーシャル・SMS/メール認証、共通例外処理 |
| プランナー | 共同計画表(編集権・自動保存・スナップショット・投票)、AIレポート、AIコンパニオンキャラクター |
| AI推薦・チャットボット | venue_type判定、天気による再推薦、アクティビティ履歴推薦、AIクレジット |
| コミュニティ | 掲示板・コメント・フォロー・チャット・DM・ユーザーブロック |
| マイページ | プロフィール・ヒートマップ・称号・アカウント管理・地域マッピング、マイトリップ (my_trips) |

## 担当範囲: 認証モジュール

会員登録・ログイン・ソーシャルログイン・本人認証・共通例外処理を設計・実装しました。

**担当範囲**: `auth/`(ログイン成功/失敗ハンドラ、`LoginAttemptService`、`CustomUserDetailsService`、OAuth2連携)、`EmailVerificationService` / `PhoneVerificationService`、`GlobalExceptionHandler` / `ErrorCode`、`SecurityConfig`

### 1. 本人認証コードの不正利用対策

- **Situation**: 会員登録時のメール認証コードは、総当たり・連続送信・再利用といった攻撃を受けやすい。
- **Task**: 認証コードの発行と検証を、安全かつ運用しやすい形で実装する。
- **Action**:
  - コードは `SecureRandom` で生成し、**BCrypt等のハッシュ(`PasswordEncoder`)のみをDBに保存**(DB流出時にコードが読めない)。
  - 有効期限5分、再送クールダウン30秒、1日の送信上限5回、検証試行上限5回を設定化(`app.verification.*`)。
  - 登録の最終送信時に `consumeVerified` で **クライアントの値を信用せずDBで再検証し、即座に使用済み化**(再利用・認証スキップの防止)。
  - 期限切れレコードを毎日04:05のバッチで削除。ログにはメールアドレスをマスクして出力。
- **Result**: 総当たり・連投・再利用の経路を塞いだ。`EmailVerificationServiceTest` などでロジックを検証済み。

### 2. SMS認証からメール認証への切り替え

- **Situation**: 当初はSMS認証(Aligo)を想定したが、事業者登録番号がないと実運用が難しいと判明した。
- **Task**: 設計を大きく崩さず、認証チャネルをメールへ切り替える。
- **Action**: `EmailSender` / `SmsSender` を**インターフェース化**し、`log`(ローカル)/ `smtp`(本番)を設定で切り替え可能にした。`EmailVerificationService` は `PhoneVerificationService` と同一構造にして、**ポリシー設定(コード長・有効時間・クールダウン)を共有**した。
- **Result**: 切り替えを小さな変更で完了。キーなしでも `log` プロバイダーでローカル開発・デモが可能(認証コードをコンソールに出力)。

### 3. ログイン失敗ロックと既知の制約

- **Situation**: パスワード総当たりへの対策が必要だった。
- **Task**: 連続失敗でアカウントを一時ロックする。
- **Action**: `LoginAttemptService` を実装。5回連続失敗で10分ロック(設定化)。`ConcurrentHashMap#compute` で**カウント更新をアトミック化**し、メールアドレスは小文字化・trimして正規化。
- **Result / 今後の課題**: 動作はテストで検証済み。ただし**状態をサーバーメモリに保持するため、複数インスタンス構成では共有できない**。スケールアウト時は Redis への移行が必要と認識しており、コードにも明記している。

### 4. ソーシャルログインの共通化とアカウント保護

- **Situation**: Google・Naver(OAuth2)と LINE(OIDC)で、アカウント連携/新規登録の判定が分かれる。
- **Task**: 提供元ごとの差異があっても、停止アカウントや重複メールの扱いが食い違わないようにする。
- **Action**: 判定ロジックを `SocialLoginResolver` に集約。連携済みなら停止アカウントを拒否、未連携でメールが既存ユーザーと重複する場合は拒否(アカウント乗っ取りの防止)、新規ならセッションに保持して追加情報入力へ誘導。
- **Result**: 3プロバイダーで同一ルールを適用。`SocialLoginResolverTest` などで検証。

### 5. 共通例外処理

- **Action**: `BusinessException` + `ErrorCode` による統一エラー体系と `GlobalExceptionHandler` を実装。**APIリクエストにはJSON、画面リクエストにはエラーページ**と、リクエスト種別に応じて応答を切り替えた。
- **Result**: 他モジュール(プランナー・コミュニティ等)も同じ例外体系を利用でき、エラー応答の形式が統一された。

<!-- TODO(本人確認): 実測できる数値(テスト件数・カバレッジ、API応答時間など)が分かれば Result に追記すると説得力が増します。 -->

## 開発ルール

- **ブランチ**: 機能ごとに `feature/*` ブランチで作業し、PRで `main` にマージ (ローカルから `main` への直接pushは禁止)。
- **スキーマ**: `schema.sql` を単一の原本とし、番号付きマイグレーションファイルで管理。マージ前にコンパイルを確認。
- **機密情報**: DB認証情報・OAuth secret・外部APIキーは `application-local.yml` (Git管理外) または環境変数でのみ注入。
- **アップロードファイル**: `${user.home}/tanoshimi-uploads` (プロジェクトフォルダの外)。
