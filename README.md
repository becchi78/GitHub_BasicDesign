# GitHub Organization 基本設計書

## 1. 概要

### 1.1 目的

本設計書は、プロジェクトにおける GitHub Organization の利用方針および運用ルールを定義することを目的としています。この設計書に基づいて開発プロセスを標準化し、効率的なチーム開発とコード品質の向上を実現します。

本書で定義する内容は以下の通りです：

- チーム構造とロールの定義
- 権限管理とアクセス制御
- レビュープロセスとワークフロー
- セキュリティとコンプライアンスの基準

### 1.2 対象範囲

本設計書、GitHub Enterprise Cloud における Organization の利用に関する基本的な設計を、組織構造、権限管理、リポジトリ運用、開発ワークフロー、品質管理の各側面について規定します。

CI/CD パイプライン、コーディングルールなどは別途定める開発ガイドラインにて規定するものとし、本設計書の対象外とします。
また、GitHub Enterprise の設計についても本設計の対象外とします。GitHub Enterprise Cloud と GitHub Organization の関係を以下の図に示します。

```mermaid
graph TD
    subgraph GitHub Enterprise Cloud

    subgraph Organization B
      RepositoryB-1
      RepositoryB-2
    end

    subgraph Organization A
      RepositoryA-1
      RepositoryA-2
    end
    end
```

Git リポジトリは Organization 単位で管理され、プロジェクト単位に Organization を割り当て権限管理をすることで、お互いのプロジェクトのリポジトリを参照できないようにします。

### 1.3 対象読者

本設計書は、以下の役割を持つメンバーを対象としています：

| メンバー         | 役割                                                          |
| ---------------- | ------------------------------------------------------------- |
| オーナー         | GitHub の管理者、プロジェクトマネージャー、プロダクトオーナー |
| チームリーダー   | チームの責任者                                                |
| シニアエンジニア | 上級開発エンジニア                                            |
| エンジニア       | 開発エンジニア                                                |

### 1.4 用語定義

| 用語            | 説明                                                               |
| --------------- | ------------------------------------------------------------------ |
| Organization    | GitHub の組織アカウント。複数のリポジトリやチームを管理する単位    |
| Repository      | ソースコードやドキュメントの格納場所。バージョン管理される最小単位 |
| Team            | Organization 内のグループ単位。権限やアクセス制御の管理単位        |
| Branch          | リポジトリ内のコード分岐。並行開発を可能にする機能                 |
| Pull Request    | コードレビューと変更の統合プロセス                                 |
| Issue           | タスク・バグ・要望の管理単位                                       |
| Review          | コードやドキュメントのレビュープロセス                             |
| Approve         | レビュー完了後の承認                                               |
| Merge           | ブランチの統合作業                                                 |
| Protection Rule | ブランチの保護設定                                                 |

## 2. Organization 設計

### 2.1 メンバーの役割定義

Organization に所属するメンバーの役割を以下のように定義します。

| メンバー         | 主な役割                                                                 |
| ---------------- | ------------------------------------------------------------------------ |
| オーナー         | システム全体の品質保証、本番環境の管理                                   |
| リーダー         | アーキテクチャ設計、チーム開発方針の策定、コードの最終承認               |
| シニアエンジニア | エンジニアの役割に加え、技術的判断、コードレビュー、エンジニアの技術支援 |
| エンジニア       | 機能実装、ユニットテスト作成、ドキュメント作成                           |

### 2.2 Team 構成

GitHub における Orgnization 内の Team 機能を使用し、以下の階層構造で権限管理を行います。

```mermaid
graph TD
   subgraph Organization
      A[Oweners] --> B[Readers]
      A --> C[Readers]

   subgraph Infrastructure Team
      B --> D[Seniors]
      D --> E[Engineers]
   end

   subgraph Application Team
      C --> F[Seniors]
      F --> G[Engineers]
   end
   end
```

（補足）Application と Infrastructure の Team には具体的な権限を付与せず、論理的なグループとしてのみ使用します。

各 Team に所属するメンバーを以下に示します。

| Team                     | 所属メンバー                     |
| ------------------------ | -------------------------------- |
| Owners                   | オーナー                         |
| Application              | アプリチームのメンバー           |
| Application/Readers      | アプリチームのリーダー           |
| Application/Seniors      | アプリチームのシニアエンジニア   |
| Application/Engineers    | アプリチームのエンジニア         |
| Infrastructure           | インフラチームメンバー           |
| Infrastructure/Readers   | インフラチームのリーダー         |
| Infrastructure/Seniors   | インフラチームのシニアエンジニア |
| Infrastructure/Engineers | インフラチームのエンジニア       |

この組織構造により、以下を実現します。

1. 柔軟なメンバー管理

   - 個人を Team に所属させることで、個人の指定なしで権限管理が可能
   - メンバーの昇格時は Team の所属を変更するだけで権限が自動的に更新
   - 退職時は Team から削除するだけで全ての権限が削除

2. 段階的な権限付与
   - Engineers: 基本的な開発権限
   - Seniors: レビュー承認権限
   - Leaders: 環境固有の承認権限

### 2.3 権限定義

GitHub における権限は以下の 3 つのレベルに分かれています。

- Organization
- Team
- Repository

それぞれのレベルにはロールがあり、以下でそのロールを示します。

#### Organization レベルのロール

Organization 全体に対する権限を定義するロールです。

| ロール | 説明                                                                          |
| ------ | ----------------------------------------------------------------------------- |
| Owner  | Organization の管理者権限。全ての設定変更、メンバー管理、支払い管理などが可能 |
| Member | Organization のメンバーとして基本的な操作が可能                               |

#### GitHub の Team レベルのロール

Team 内での権限を定義するロールです。

| ロール     | 説明                                                            |
| ---------- | --------------------------------------------------------------- |
| Maintainer | チームの管理者権限。メンバーの追加・削除、Team 設定の変更が可能 |
| Member     | チームのメンバーとして基本的な操作が可能                        |

#### GitHub の Repository レベルロール

リポジトリに対する権限を定義するロールです。

| ロール   | 説明                                                       |
| -------- | ---------------------------------------------------------- |
| Admin    | リポジトリの管理者権限。全ての設定変更とアクセス管理が可能 |
| Maintain | リポジトリの管理（設定変更を除く）とコードの変更が可能     |
| Write    | コードの変更と Pull Request の作成が可能                   |
| Triage   | Issue や Pull Request の管理が可能（コード変更不可）       |
| Read     | 読み取りのみ可能                                           |

### 2.4 承認権限の設定

#### Team へのロール割り当て

各レベルのロールを Team に割り当てることで Team に対して適切な権限設定を行います。

| Team                     | Organization Role | Team Role  | Repository Role |
| ------------------------ | ----------------- | ---------- | --------------- |
| Owners                   | Owner             | -          | -               |
| Application              | Member            | -          | -               |
| Application/Leaders      | Owner             | Maintainer | Admin           |
| Application/Seniors      | Member            | Maintainer | Admin           |
| Application/Engineers    | Member            | Member     | Write           |
| Infrastructure           | Member            | -          | -               |
| Infrastructure/Leaders   | Owner             | Maintainer | Admin           |
| Infrastructure/Seniors   | Member            | Maintainer | Admin           |
| Infrastructure/Engineers | Member            | Member     | Write           |

#### ブランチ保護ルール

各ブランチには以下の保護ルールを設定します。

| ブランチ   | 必要な承認             | マージ権限       | 追加設定                         |
| ---------- | ---------------------- | ---------------- | -------------------------------- |
| production | Owners の承認          | Owners のみ      | 承認必須、ステータスチェック必須 |
| staging    | Leaders の承認         | Leaders のみ     | 承認必須、ステータスチェック必須 |
| develop    | Seniors/Leaders の承認 | Write 権限保持者 | 承認必須、ステータスチェック必須 |

#### Code Owners の設定

各ブランチに対して.github/CODEOWNERS の設定をすることで、ブランチ全体に対する制限を設定します。

```text
# デフォルトのCode Owner設定
*                   @Organization/Infrastructure/Seniors @Organization/Application/Seniors
                    @Organization/Infrastructure/Leaders @Organization/Application/Leaders

# productionブランチ全体の所有者設定
production          @Organization/Owners

# stagingブランチ全体の所有者設定
staging             @Organization/Infrastructure/Leaders @Organization/Application/Leaders

# developブランチ全体の所有者設定
develop             @Organization/Infrastructure/Seniors @Organization/Application/Seniors
                    @Organization/Infrastructure/Leaders @Organization/Application/Leaders
```

#### 権限の優先順位

GitHub における権限制御は、以下の順序で優先されます。

1. Branch Protection Rules / CODEOWNERS（最優先）

   - ブランチごとの保護設定
   - マージ制限、承認要件
   - コードオーナーの指定
   - 例：production ブランチへのマージには Owner の承認が必須

2. Repository Level（中間）

   - リポジトリごとの権限設定
   - Repository Role（Admin, Maintain, Write 等）による基本権限
   - 例：Admin 権限を持っていても、Branch Protection の制限は上書きできない

3. Organization Level（最低位）
   - Organization 全体の設定
   - Team や Member の基本権限
   - 例：Organization Member であっても Repository 権限がなければアクセス不可

この優先順位により、上位の設定が下位の設定を上書きします。例えば、Repository Role で Admin 権限を持つユーザーでも、Branch Protection や CODEOWNERS の設定により、特定のブランチへのマージや承認が制限されます。

### 2.5 操作権限マトリクス

| 操作権限                  | オーナー | リーダー | シニアエンジニア | エンジニア |
| ------------------------- | :------: | :------: | :--------------: | :--------: |
| **Organization Level**    |          |          |                  |            |
| Organization 設定変更     |    ✅    |    ✅    |        ❌        |     ❌     |
| Team 作成・削除           |    ✅    |    ✅    |        ❌        |     ❌     |
| メンバー管理              |    ✅    |    ✅    |        ❌        |     ❌     |
| Billing 管理              |    ✅    |    ✅    |        ❌        |     ❌     |
| **Team Level**            |          |          |                  |            |
| Team 設定変更             |    ✅    |   ✅\*   |       ✅\*       |     ❌     |
| Team メンバー管理         |    ✅    |   ✅\*   |       ✅\*       |     ❌     |
| **Repository Level**      |          |          |                  |            |
| リポジトリ作成            |    ✅    |    ✅    |        ✅        |     ❌     |
| リポジトリ設定変更        |    ✅    |    ✅    |        ✅        |     ❌     |
| ブランチ保護設定          |    ✅    |    ✅    |        ✅        |     ❌     |
| Webhooks 設定             |    ✅    |    ✅    |        ✅        |     ❌     |
| セキュリティ設定          |    ✅    |    ✅    |        ✅        |     ❌     |
| PR のマージ（production） |    ✅    |    ✅    |        ❌        |     ❌     |
| PR のマージ（staging）    |    ✅    |    ✅    |        ✅        |     ❌     |
| PR のマージ（develop）    |    ✅    |    ✅    |        ✅        |     ✅     |
| PR の承認（production）   |    ✅    |    ✅    |        ❌        |     ❌     |
| PR の承認（staging）      |    ✅    |    ✅    |        ✅        |     ❌     |
| PR の承認（develop）      |    ✅    |    ✅    |        ✅        |     ❌     |
| PR のレビュー             |    ✅    |    ✅    |        ✅        |     ❌     |
| Issue 管理                |    ✅    |    ✅    |        ✅        |     ✅     |
| コードの Push             |    ✅    |    ✅    |        ✅        |     ✅     |

- ✅\* は「自身が所属する Team のみ可能」を示します。

## 3 保護ブランチの設定

GitHub のブランチ保護機能を利用して、ブランチ（production、staging、develop）への変更を制御します。この設定により、意図しない変更の防止と、適切なレビュープロセスの実施を強制します。

### 3.1 ブランチ保護の基本方針

1. 直接的な変更の防止

   保護対象ブランチへの直接的なプッシュを禁止し、すべての変更を Pull Request を通じて行うことを強制します。
   これにより、すべての変更に対してレビューと CI チェックを確実に実施できます。

2. 段階的な承認プロセス

   環境ごとに異なる承認要件を設定し、変更の重要度に応じた承認プロセスを実現します。
   production 環境への変更は最も厳格な承認プロセスを必要とします。

3. 品質チェックの強制

   develop 保護ブランチで、マージ前のステータスチェック（CI）の通過を必須とします。
   コードの品質基準を満たすことを保証します。

### 3.2 保護ブランチの設定

#### 具体的な保護設定

以下に、GitHub Repository の設定画面における具体的な設定項目と設定値を示します。

【production ブランチ】

```markdown
Required Reviews

- ✓ Require a pull request before merging
  - ✓ Require approvals (1 人)
  - ✓ Dismiss stale pull request approvals when new commits are pushed
  - ✓ Require review from Code Owners
  - ✓ Restrict who can dismiss pull request reviews (Owners only)
- ✓ Require status checks to pass before merging
  - ✓ Require branches to be up to date before merging
  - Required status checks:
    - continuous-integration/jenkins/pr
    - security-scan
    - lint-check
- ✓ Require conversation resolution before merging
- ✓ Include administrators
- ✓ Restrict who can push to matching branches
  - Allow specified actors (Owners only)
```

【staging ブランチ】

```markdown
Required Reviews

- ✓ Require a pull request before merging
  - ✓ Require approvals (1 人)
  - ✓ Dismiss stale pull request approvals when new commits are pushed
  - ✓ Require review from Code Owners
  - ✓ Restrict who can dismiss pull request reviews (Leaders only)
- ✓ Require status checks to pass before merging
  - ✓ Require branches to be up to date before merging
  - Required status checks:
    - continuous-integration/jenkins/pr
    - lint-check
- ✓ Require conversation resolution before merging
- ✓ Include administrators
- ✓ Restrict who can push to matching branches
  - Allow specified actors (Leaders only)
```

【develop ブランチ】

```markdown
Required Reviews

- ✓ Require a pull request before merging
  - ✓ Require approvals (1 人)
  - ✓ Dismiss stale pull request approvals when new commits are pushed
  - ✓ Require review from Code Owners
- ✓ Require status checks to pass before merging
  - ✓ Require branches to be up to date before merging
  - Required status checks:
    - continuous-integration/jenkins/pr
    - lint-check
- ✓ Require conversation resolution before merging
- ✓ Include administrators
```

#### 補足事項

1. レビュー承認の破棄

   - 新しいコミットがプッシュされた際は、既存の承認を自動的に破棄します。
   - これにより、変更後のコードに対する再レビューを強制します。

2. 会話の解決要件

   - すべての保護ブランチで、PR のレビューコメントの解決を必須としています。
   - これにより、レビューで指摘された問題の確実な対応を促します。

3. 管理者への適用
   - 管理者（Admin 権限保持者）に対しても同じ制限を適用します。
   - これにより、誤操作による意図しない変更を防ぎます。

## 4. Pull Request 設計

Pull Request（PR）を用いた開発ワークフローについて定義します。

### 4.1 Pull Request とは

Pull Request（以下、PR）は、コードの変更内容をチームメンバーと共有し、レビューを受けるための GitHub の機能です。PR を通じて、コードの品質維持、知識の共有、およびチーム内のコラボレーションを促進します。

PR では以下の作業を行います。

- コードの差分の確認
- レビューコメントの投稿
- コードに対する提案
- 議論のスレッド管理
- CI による自動テストの実行

### 4.2 Pull Request フロー

develop ブランチに変更を merge する際には以下のフローで作業を行います。

1. ブランチの作成

   開発者は作業用のブランチを作成します。ブランチ名は定められた命名規則に従います。

2. コードの実装

   作業用ブランチで必要な実装を行います。このとき、以下の点に注意します。

   - コミットは論理的な単位で行う
   - コミットメッセージは明確に記述する
   - 開発ガイドラインに従った実装を行う

3. コードの Push

   実装したコードをリモートリポジトリにプッシュします。この時点で以下が自動実行されます。

   - GitHub Actions による CI の実行
     - 静的解析
     - 単体テスト
     - リンターチェック
   - CI の結果は GitHub 上で確認可能

   CI によるテストにパスしないと PR の作成が行えません。

4. PR の作成

   CI が成功したことを確認後、以下の手順で PR を作成します：

   - 適切なブランチをマージ先として選択
   - PR template に従って内容を記述
   - レビュアーを 1 名以上指定（該当環境に応じた権限を持つメンバー）
   - 関連する Issue をリンク

5. レビュープロセス
   以下の流れでレビューを実施します：

   - レビュアーによるコードレビュー
   - レビューコメントへの対応
   - 必要に応じてコードの修正

   すべてのレビューコメントが解決され、必要な承認が得られるまでこのプロセスを繰り返します。

6. マージの実行
   必要な承認と CI チェックが完了したら、PR 作成者が以下を確認してマージを実行します：
   - 必要な承認が得られていること
   - すべてのレビューコメントが解決していること
   - コンフリクトが発生していないこと

develop から staging、staging から production に merge を行う際には、4 ～ 6 を実施します。

### 4.3 Pull Request Template

PR template の内容を以下に規定します。

```markdown
## 概要

<!-- 変更の目的と概要を記載してください -->

## 変更内容

<!-- 具体的な変更内容を箇条書きで記載してください -->

## テスト内容

<!-- 実施したテストの内容を記載してください -->
<!-- エビデンスは直接貼り付けるかリンクを付けてください -->

## レビュー項目

<!-- レビュアーに特に確認して欲しい点があれば記載してください -->

## 関連チケット

<!-- 関連するチケット番号があれば記載してください -->
```

以下に PR の例を示します。

```
## 概要

S3バケット監視用Lambda関数に、CloudWatch Logsへの詳細なログ出力機能を追加しました。これにより、ファイル監視とSNS通知の実行状況をより詳細に把握できるようになり、トラブルシューティングが容易になります。

## 変更内容

- Lambda関数（`s3_monitor.py`）に以下のログ出力を追加：
  - S3イベント受信時のイベント内容（バケット名、オブジェクトキー、イベントタイプ）
  - SNSメッセージ送信前の通知内容
  - 処理の開始・完了タイミング
  - エラー発生時の詳細情報

- IAMロール設定の更新：
  - CloudWatch Logsへの書き込み権限（`logs:CreateLogStream`、`logs:PutLogEvents`）を追加

## テスト内容

1. ローカル環境でのテスト

# テストケース実行結果
test_s3_event_logging: PASS
test_sns_message_logging: PASS
test_error_logging: PASS

2. AWS環境での動作確認
- S3バケットへのファイルアップロード
- CloudWatch Logsでのログ確認

START RequestId: 123e4567-e89b-12d3-a456-426614174000
2024-01-01T00:00:00.000Z [INFO] Received S3 event for bucket: my-bucket, key: test.txt
2024-01-01T00:00:01.000Z [INFO] Sending SNS notification: File test.txt uploaded to my-bucket
2024-01-01T00:00:02.000Z [INFO] SNS notification sent successfully
END RequestId: 123e4567-e89b-12d3-a456-426614174000

## レビュー項目

- ログレベルの適切性（INFO/ERROR）の選定
- ログ出力による関数の実行時間への影響
- 出力内容に機密情報が含まれていないことの確認
- エラー時のログ出力フォーマットの妥当性

## 関連チケット

INFRA-789: Lambda関数のログ出力機能強化
```

### 4.4 レビュープロセス

各環境に応じたレビューレビュアーを以下のように定義します。

| 環境       | レビュー要件                                 | レビュー観点                                                       |
| ---------- | -------------------------------------------- | ------------------------------------------------------------------ |
| develop    | シニアエンジニアまたはリーダーによるレビュー | 実装の妥当性、テストの十分性、アーキテクチャ整合性                 |
| staging    | シニアエンジニアまたはリーダーによるレビュー | develop 環境でのテスト結果の妥当性、性能への影響、セキュリティ考慮 |
| production | オーナーまたはリーダーによるレビュー         | staging 環境でのテスト結果の妥当性、リリース要件充足               |

レビューについては 6 章の「レビュー方針」で詳しく記載します。

## 5 Issue 管理

### 5.1 Issue とは

Issue は、プロジェクトにおけるタスク、バグ、機能要望などを管理するための GitHub の機能です。チケットとしても使用され、以下のような用途で活用されます。

- バグ報告の管理
- 機能要望の記録
- タスクの進捗管理
- 議論のスレッド管理

Issue には以下の特徴があります。

- 一意の番号が付与される
- ラベルによる分類が可能
- 担当者の割り当てが可能
- PR との相互リンクが可能
- コメントによる議論が可能

### 5.2 Issue template

Issue template の内容を以下に規定します。

```markdown
## バグ報告テンプレート

### 発生環境

- 環境：
- ブラウザ/バージョン：

### 再現手順

1.
2.

### 期待動作

### 実際の動作

### スクリーンショット（あれば）

---

## 機能要望テンプレート

### 概要

### 目的・背景

### 提案内容

### 期待される効果
```

### 5.3 Issue 運用ルール

プロジェクトでは、効率的なタスク管理とコミュニケーションを実現するため、以下のように Issue を運用します。

1. Issue 作成のタイミング
   以下の場合に Issue を作成します。

   - バグを発見したとき
   - 新機能や改善の提案があるとき
   - 定義された作業単位の実装を開始するとき

2. Issue 管理のプラクティス
   効率的な管理のため、以下の運用ルールを設けます。

   - 作業開始時には担当者を割り当てる
   - 進捗状況は適宜コメントで更新する
   - 関連する PR は Issue にリンクする
   - クローズ時は対応結果を必ず記載する

3. クローズの条件
   以下の条件を満たした際に Issue をクローズします：

   バグ報告の場合：

   - 修正が完了し、テストで確認できた場合
   - 仕様通りの動作と判明した場合
   - 再現性が確認できず、報告者も同意した場合

   機能要望の場合：

   - 実装が完了し、テストで確認できた場合
   - 実装しないことが決定し、関係者の同意を得た場合
   - 別の方法で要件が満たされることになった場合

   タスクの場合：

   - 作業が完了し、成果物のレビューが完了した場合
   - タスクが不要となり、関係者の同意を得た場合

# 以下念のための保存

## 4. レビュー方針

### 3.1 プルリクエスト作成ガイドライン

以下のテンプレートを使用して PR を作成します：

```markdown
## 概要

変更内容の概要を記載してください。

## 変更内容

- 具体的な変更点をリストアップしてください
- できるだけ箇条書きで記載してください

## 影響範囲

この変更による影響範囲を記載してください。

## テスト内容

- [ ] 実施したテストをチェックリスト形式で記載
- [ ] ユニットテストの実施
- [ ] 動作確認の実施

## 関連情報

関連する PR、Issue、ドキュメントへのリンクを記載してください。
```

#### 説明文の記載ルール

| 項目       | 記載内容             | 例                                   |
| ---------- | -------------------- | ------------------------------------ |
| 概要       | 変更の目的と全体像   | 「ログイン機能のパフォーマンス改善」 |
| 変更内容   | 技術的な変更点       | 「セッション管理を Redis に移行」    |
| 影響範囲   | 変更の影響が及ぶ機能 | 「認証機能全般、特にセッション周り」 |
| テスト内容 | 実施した検証項目     | 「負荷テスト、セッション永続化確認」 |

#### 関連チケットの紐付け

PR の説明文に以下の形式で記載します：

```text
Related to: #123
Fixes: #456
```

### 3.2 レビュー実施ガイドライン

#### レビュアー指名のルール

- 必要レビュアー数：最低 1 名
- 指名基準：
  - インフラ変更：Infrastructure Team のシニアエンジニア以上
  - アプリケーション変更：Application Team のシニアエンジニア以上
  - 共通部分の変更：両チームのレビュアーが必要

#### レビュー期間

| 変更の種類     | 標準期間     | 緊急時の期間 |
| -------------- | ------------ | ------------ |
| 軽微な変更     | 1 営業日以内 | 2 時間以内   |
| 通常の機能追加 | 2 営業日以内 | 4 時間以内   |
| 大規模な変更   | 3 営業日以内 | 1 営業日以内 |

#### レビューコメントの記載方法

```markdown
[種類: 提案/問題点/質問]
指摘内容を具体的に記載

理由：
なぜその指摘が必要なのかの説明

提案：
改善案がある場合は具体的に記載
```

### 3.3 フィードバック手順

#### コメントの種類と対応

| コメントの種類 | 内容             | 必須対応 |
| -------------- | ---------------- | -------- |
| Must Fix       | 重大な問題の指摘 | 必須     |
| Suggestion     | 改善提案         | 任意     |
| Question       | 実装に関する質問 | 回答必須 |
| Nitpick        | 軽微な指摘       | 任意     |

#### 指摘事項への対応方法

1. すべてのコメントに返信する
2. 対応完了後は Resolve する
3. 大きな修正が必要な場合は新しいコミットを作成
4. 軽微な修正は既存のコミットに追加（`git commit --amend`）

### 3.4 承認フロー

#### 環境別の承認ルール

```markdown
Develop 環境

- シニアエンジニアまたはチームリーダーの承認が必要
- CI/CD のすべてのチェックがパス
- 必要なレビューコメントへの対応完了

Staging 環境

- チームリーダーの承認が必要
- Develop 環境での動作確認完了
- 結合テストのパス

Production 環境

- オーナーの承認が必要
- Staging 環境での検証完了
- セキュリティチェックのパス
```

#### 承認基準

| 確認項目     | 基準                                                                 |
| ------------ | -------------------------------------------------------------------- |
| コードの品質 | - コーディング規約に準拠、適切なテストの実装、パフォーマンスへの考慮 |
| ドキュメント | - 必要な設計書の更新、API ドキュメントの更新、変更履歴の記録         |
| セキュリティ | - 脆弱性の未使用、適切な認証・認可、機密情報の非公開                 |

#### 差し戻し時の対応

1. レビュアーは差し戻しの理由を明確に記載
2. 開発者は以下の手順で対応：
   - 指摘事項の確認と理解
   - 修正方針の策定と共有
   - 修正の実施と再テスト
   - 変更内容の説明を追記
3. 再レビューの依頼
