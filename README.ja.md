# タノシミ (tanoshimi / 타노시미)

[한국어](README.md) | **日本語**

[![CI](https://github.com/good200005/tanoshimi/actions/workflows/ci.yml/badge.svg)](https://github.com/good200005/tanoshimi/actions/workflows/ci.yml)

> 日韓交流旅行のパーティーマッチング + リアルタイム共同プランニング + AI旅行おすすめサービス

**「楽しみ(タノシミ)」= 期待・ワクワク。** 旅行に*一緒に行く人探し*から*プラン作成*、*帰ってからの記録*までを、ひとつのサービスでつなぎます。

## デモ

| | リンク | 説明 |
|---|---|---|
| **本番サービス** | https://tanoshimi.onrender.com | ログイン・パーティー・プランナー・チャットなど全機能が動作 (Render + TiDB Cloud) |
| **静的プレビュー** | https://good200005.github.io/tanoshimi/ | サーバーなしで画面のみ確認 (GitHub Pages、機能は動作しません) |

> [!IMPORTANT]
> **本番サービスは無料プランで運用しているため、初回アクセス時は起動に約1分かかります。**
> 「SERVICE WAKING UP」の画面が表示されたら、ページを閉じずにそのままお待ちください(起動後は自動でログイン画面に切り替わります)。起動後は通常の速度で動作します。
> すぐに確認したい場合は、下記のデモ動画または静的プレビュー(即時表示、機能は動作しません)をご覧ください。

- デモ動画: チームで作成中(完成後にリンクを追加します)。
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

### 0. Docker で一発起動 (最短)

Docker があれば、MySQL の準備から起動までコマンド1つで完了します(リポジトリルートで実行)。

```bash
docker compose up --build   # http://localhost:8080
```

- 初回起動時に `schema.sql` と `data.sql`(デモデータ、全アカウントのパスワード: `Test1234!`)が自動で適用されます。
- 外部連携は `mock`、メール認証コードはアプリのログに出力されます (`docker compose logs -f app`)。
- DB を初期化するには `docker compose down -v` を実行します。

Docker を使わず手動で構築する場合は、以下の 1〜3 に従ってください。

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
├── docker-compose.yml          # MySQL + アプリの一発起動
├── .github/workflows/          # CI (ci.yml), CodeQL, Pages
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
| [07. 構成図・認証ER図・主要フロー](tanoshimi/docs/07-구성도-ER도.md) | システム構成図、認証モジュールのER図、パスワード再発行フロー、例外処理の分岐 (Mermaid) |
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

### 2. パスワード再発行フローの設計(チームで議論して決定)

- **Situation**: 「パスワードを忘れた」機能は、実装方法によっては**他人のアカウントを乗っ取る・使用不能にする経路**になる。
- **Task**: 安全で、ユーザーにとって自然な再発行フローを決める。
- **Action**: 実装前にフロー上の分岐点を整理し、チームメンバーとレビューして次の方針を決定した。
  - **本人確認を必須化**: 再発行ページへ直接遷移させず、メールの認証コード(`VerificationPurpose.find_password`)を通過した場合のみ臨時パスワードを発行。もともとスキーマに定義だけあって未使用だった用途を活用し、既存の認証コード基盤を再利用した。
  - **臨時パスワードの扱い**: 私は「ログインした時点で正式パスワードになる」案を考えたが、**「再発行を要求した時点で旧パスワードは信用できない」**というレビューを受け、発行と同時に旧パスワードを無効化する方針に変更(多くのサービスの一般的な設計)。
  - **強制変更**: `UserEntity.mustChangePassword` フラグを立て、臨時パスワードでのログイン後は `LoginSuccessHandler` がパスワード変更画面へ強制誘導。変更完了時にセッション上のフラグも即時更新(更新しないと再ログインまで変更モーダルが出続ける)。
  - ソーシャル専用アカウントは再発行の対象外(`SOCIAL_ACCOUNT_NO_PASSWORD`)。
- **Result**: 「メールアドレスを知っているだけで他人のパスワードを無効化できる」という初期実装の穴を、実装段階で塞げた。**仕様の分岐点を言語化→相談→決定する流れ**を実践した。

### 3. SMS認証からメール認証への切り替え

- **Situation**: 当初はSMS認証(Aligo)を想定したが、事業者登録番号がないと実運用が難しいと判明した。
- **Task**: 設計を大きく崩さず、認証チャネルをメールへ切り替える。
- **Action**: `EmailSender` / `SmsSender` を**インターフェース化**し、`log`(ローカル)/ `smtp`(本番)を設定で切り替え可能にした。`EmailVerificationService` は `PhoneVerificationService` と同一構造にして、**ポリシー設定(コード長・有効時間・クールダウン)を共有**した。
- **Result**: 切り替えを小さな変更で完了。キーなしでも `log` プロバイダーでローカル開発・デモが可能(認証コードをコンソールに出力)。

### 4. ログイン失敗ロックと既知の制約

- **Situation**: パスワード総当たりへの対策が必要だった。
- **Task**: 連続失敗でアカウントを一時ロックする。
- **Action**: `LoginAttemptService` を実装。5回連続失敗で10分ロック(設定化)。`ConcurrentHashMap#compute` で**カウント更新をアトミック化**し、メールアドレスは小文字化・trimして正規化。
- **Result / 今後の課題**: 動作はテストで検証済み。ただし**状態をサーバーメモリに保持するため、複数インスタンス構成では共有できない**。スケールアウト時は Redis への移行が必要と認識しており、コードにも明記している。

### 5. ソーシャルログイン × 既存アカウントの重複問題(3案を比較して決定)

- **Situation**: 通常の会員登録で作成済みのユーザーが、そのことを忘れてソーシャルログインで再登録すると、**同一人物に複数アカウントができる**。
- **Task**: 重複を防ぎつつ、アカウント乗っ取りのリスクを作らない方針を決める。
- **Action**: 3案のメリット・デメリットを整理してチームに提案した。
  | 案 | 内容 | 評価 |
  |---|---|---|
  | 1. 何もしない | 別アカウントとして扱う | 最も簡単だが、通報・BANの回避に悪用できる |
  | 2. 自動統合 | メールアドレスが一致したら自動で紐付け | 便利だが、提供元ごとのメール信頼性が異なり**他人のアカウントに侵入される恐れ** |
  | **3. 手動連携(採用)** | ログイン後にマイページ > アカウント管理 > ソーシャル連携で本人が紐付け | 現実的かつ安全 |
  - 議論の結果、案3を採用。コードを確認すると連携機能は実装済みで、不足は**案内だけ**だった。新機能は作らず、`OAuth2FailureHandler` の `EMAIL_ALREADY_USED` のメッセージを「ログイン後、マイページ > アカウント管理 > ソーシャル連携で連携してください」に改善した。
  - 3つのプロバイダー(Google・Naver・LINE)で判定が食い違わないよう、ロジックを `SocialLoginResolver` に集約(停止アカウントの拒否、メール重複の拒否、新規ユーザーは追加情報入力へ誘導)。
- **Result**: セキュリティ上のリスクを増やさず、**既存機能を活かして最小の変更で解決**した。`SocialLoginResolverTest` などで検証。

### 6. 共通例外処理 — 想定外の例外がスタックトレースとして露出する問題の発見と修正

- **Situation**: `BusinessException` + `ErrorCode` による統一エラー体系と `GlobalExceptionHandler` を実装していたが、適用範囲が `@RestController` のみだった。
- **Problem**: 画面用 `@Controller` の中に `@ResponseBody` のAPIメソッドを混在させたコントローラーが6つあり(`PostController`、`PlannerController`、`PartyController`、`PartyRoomController`、`RecommendationController`、`MyPageController`)、そこで例外が起きると**Spring標準のエラーページ(スタックトレース)がユーザーに露出**していた。
- **Action**: 適用範囲を全コントローラーに広げ、**`@ResponseBody` の有無(またはURIが `/api/`)で分岐**。APIはJSONで、画面は従来のエラーページ(`error/403.html` など)へ流す。この過程で以下も修正した。
  - `@ExceptionHandler` のパラメーターに `HandlerMethod` を直接受け取ると、Servlet/Securityフィルター由来の例外(ログイン経路など)でハンドラー自体が `IllegalStateException` で失敗する → 常に解決できる `HttpServletRequest` に変更。
  - 実際に2つのアカウントで動かして発見: パーティー外のユーザーがURLを直接開くと、期待する403ではなく**500エラー**になる。例外を再スローすると `ErrorCode` のステータスが失われるため。
  - **試して捨てた方法**: `@ExceptionHandler` 内で新しい例外(`ResponseStatusException`)を投げる方式は、「Failure in @ExceptionHandler」として500になる(ログで確認)。最終的に `HttpServletResponse#sendError` で応答を確定させ、コンテナ標準のエラーディスパッチ経由で `error/{status}.html` を表示させた。
- **Result**: 他モジュールも同じ例外体系を使え、API/画面の双方でエラー応答が統一された。コード上のコメントだけでは正常に見えたが、**実際に動かして初めて気づけた**問題で、「コードを読むだけでなく実機で検証する」姿勢を学んだ。

### テストと品質(実測値)

`./gradlew test` と JaCoCo で測定しました(プロジェクト全体のカバレッジは約25%のため、担当した認証モジュールのみを記載)。

| 指標 | 結果 |
|---|---|
| 全体のテスト | 213件すべて成功 (失敗・エラー・スキップ 0件) |
| 認証関連のテスト | 13クラス・67件すべて成功 |
| 認証パッケージ (`auth`) のカバレッジ | ライン 69.4% / 分岐 72.0% |
| `EmailVerificationService` | ライン 97.4% / 分岐 85.7% |
| `SocialLoginResolver` | ライン 100% / 分岐 100% |
| `LoginAttemptService` | ライン 84.2% / 分岐 83.3% |
| `UserService` (パスワード再発行を含む) | ライン 92.2% / 分岐 70.8% |
| `GlobalExceptionHandler` | ライン 77.8% / 分岐 77.3% |

※ JaCoCo はビルド設定に組み込み済みで、`./gradlew test` でレポート (`build/reports/jacoco/test/`) が生成されます。CI (GitHub Actions) でも同じテストを自動実行し、カバレッジレポートを artifact として保存します。

## 開発ルール

- **ブランチ**: 機能ごとに `feature/*` ブランチで作業し、PRで `main` にマージ (ローカルから `main` への直接pushは禁止)。
- **スキーマ**: `schema.sql` を単一の原本とし、番号付きマイグレーションファイルで管理。マージ前にコンパイルを確認。
- **機密情報**: DB認証情報・OAuth secret・外部APIキーは `application-local.yml` (Git管理外) または環境変数でのみ注入。
- **アップロードファイル**: `${user.home}/tanoshimi-uploads` (プロジェクトフォルダの外)。
