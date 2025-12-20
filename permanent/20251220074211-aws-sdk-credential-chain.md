---
id: 2025-12-20-074211
title: AWS SDK Credential Chainの動作メカニズム
created: 2025-12-20 07:42:11
updated: 2025-12-20 07:42:11
tags:
  - zettel
  - aws
  - sdk
  - authentication
  - credential-chain
  - security
source:
  type: concept
  ref: "AWS SDK documentation - Credential Provider Chain"
---

## 要約（Summary）

- AWS SDK の Credential Chain は、複数のソースから順番に認証情報を探す仕組み
- 環境変数 → 設定ファイル → コンテナ/インスタンスメタデータ の順で探索
- 最初に見つかった有効な認証情報を使用し、以降のソースは参照しない
- この仕組みにより、環境ごとに認証方法を柔軟に切り替えられる

## 本文（Body）

AWS SDK の Credential Chain（認証情報チェーン）は、アプリケーションが AWS サービスにアクセスする際の認証情報を、予め定められた優先順位で自動的に探索・選択するメカニズムである。この理解は、マルチ環境での AWS 認証設計において必須となる。

### 背景・問題意識

**認証方法の多様性と環境の違い**

AWS アプリケーションは様々な環境で動作する：
- ローカル開発環境 → `~/.aws/credentials` を使いたい
- CI/CD 環境 → 環境変数でキーを渡したい
- EC2/ECS 本番環境 → IAM ロールを使いたい

それぞれで**異なる認証方法**が必要だが、アプリケーションコードは環境ごとに書き換えたくない。

**従来の問題（Credential Chain なし）**

```go
// 悪い例：環境ごとにコード変更が必要
if os.Getenv("ENV") == "production" {
    sess = session.NewSessionWithInstanceRole()
} else if os.Getenv("ENV") == "ci" {
    sess = session.NewSessionWithEnvVars()
} else {
    sess = session.NewSessionWithSharedConfig()
}
```

### アイデア・主張

**AWS SDK の Credential Chain は、予め定義された優先順位で複数の認証ソースを自動探索し、最初に見つかった有効な認証情報を使用することで、環境ごとのコード変更なしに柔軟な認証を実現する。**

#### 標準的な Credential Chain の順序

**1. 環境変数（最優先）**
```bash
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN  # 一時的な認証情報の場合
```

**2. 共有設定ファイル**
```bash
~/.aws/credentials
~/.aws/config  # AWS_SDK_LOAD_CONFIG=true が必要（Go v1）
```

**3. コンテナ認証情報（ECS Task Role）**
```
AWS_CONTAINER_CREDENTIALS_RELATIVE_URI  # ECS が自動設定
```

**4. インスタンスメタデータ（EC2 Instance Role）**
```
http://169.254.169.254/latest/meta-data/iam/security-credentials/
```

#### 動作の原則

**1) 最初にヒットしたら停止**

```
環境変数にキーあり
  → 以降（ファイル、ロール）は参照しない
  → 環境変数のキーを使用
```

**2) 見つからなければ次へ**

```
環境変数なし
  → ~/.aws/credentials を確認
    → あればそれを使用
    → なければ ECS Task Role を確認
      → ...
```

**3) すべて失敗したらエラー**

```go
NoCredentialProviders: no valid providers in chain
```

### 内容を視覚化するMermaid図

```mermaid
graph TB
    Start["AWS SDK<br/>認証情報取得開始"] --> Check1["1. 環境変数<br/>AWS_ACCESS_KEY_ID<br/>確認"]
    
    Check1 -->|あり| Use1["✓ 環境変数を使用"]
    Check1 -->|なし| Check2["2. 共有設定ファイル<br/>~/.aws/credentials<br/>確認"]
    
    Check2 -->|あり| Check2b["AWS_SDK_LOAD_CONFIG<br/>の値確認"]
    Check2b -->|true| Use2a["✓ credentials + config<br/>を使用"]
    Check2b -->|false/未設定| Use2b["✓ credentials のみ使用"]
    
    Check2 -->|なし| Check3["3. ECS Task Role<br/>AWS_CONTAINER_CREDENTIALS_<br/>RELATIVE_URI確認"]
    
    Check3 -->|あり| Use3["✓ ECS Task Role<br/>メタデータから取得"]
    Check3 -->|なし| Check4["4. EC2 Instance Role<br/>IMDSv2エンドポイント<br/>確認"]
    
    Check4 -->|あり| Use4["✓ Instance Role<br/>メタデータから取得"]
    Check4 -->|なし| Error["❌ NoCredentialProviders<br/>エラー"]
    
    Use1 --> End["AWS API呼び出し"]
    Use2a --> End
    Use2b --> End
    Use3 --> End
    Use4 --> End
    
    style Use1 fill:#ccffcc
    style Use2a fill:#ccffcc
    style Use2b fill:#ccffcc
    style Use3 fill:#ccffcc
    style Use4 fill:#ccffcc
    style End fill:#ccffcc
    style Error fill:#ffcccc
```

### 具体例・ケース

#### ケース1: 環境ごとの自動切り替え

**同じアプリケーションコード:**
```go
// main.go（環境に依存しない）
sess := session.Must(session.NewSession())
svc := s3.New(sess)
// 認証方法は Credential Chain が自動選択
```

**ローカル開発環境:**
```bash
# ~/.aws/credentials が存在
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = xxx

# → Credential Chain がこれを使用
```

**CI/CD 環境:**
```yaml
# GitHub Actions
env:
  AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
  AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
  
# → 環境変数が最優先なので、これを使用
```

**EC2 本番環境:**
```bash
# IAM Instance Role がアタッチされている
# → ~/.aws/credentials なし、環境変数なし
# → Credential Chain が Instance Metadata を使用
```

#### ケース2: 環境変数での一時的な上書き

**通常:**
```bash
# ~/.aws/credentials
[default]
aws_access_key_id = PROD_KEY
```

**デバッグ用に別アカウントを使いたい:**
```bash
# 環境変数で一時的に上書き（Credential Chain の最優先）
export AWS_ACCESS_KEY_ID=DEBUG_KEY
export AWS_SECRET_ACCESS_KEY=debug_secret

./myapp
# → DEBUG_KEY で実行される（~/.aws/credentials は無視）

unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY
./myapp
# → 再び ~/.aws/credentials の PROD_KEY に戻る
```

#### ケース3: プロファイルの指定

**設定:**
```ini
# ~/.aws/credentials
[default]
aws_access_key_id = DEFAULT_KEY

[production]
aws_access_key_id = PROD_KEY

[development]
aws_access_key_id = DEV_KEY
```

**アプリケーション:**
```go
// プロファイルを明示的に指定
sess := session.Must(session.NewSessionWithOptions(session.Options{
    Profile: "production",
}))
// → PROD_KEY を使用

// または環境変数で指定
// export AWS_PROFILE=development
// → DEV_KEY を使用
```

#### ケース4: トラブルシューティング - Credential Chain の確認

**認証エラーが出たとき:**
```bash
# 1. 環境変数の確認
env | grep AWS

# 2. 設定ファイルの確認
cat ~/.aws/credentials
cat ~/.aws/config

# 3. ECS 環境の確認
echo $AWS_CONTAINER_CREDENTIALS_RELATIVE_URI

# 4. EC2 Instance Role の確認
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# 5. SDK のデバッグログ有効化
export AWS_SDK_LOAD_CONFIG=true
export AWS_SDK_GO_LOG=DEBUG
./myapp
```

### 反論・限界・条件

**「環境変数が最優先」の落とし穴**

意図せず環境変数が残っていると、ファイルやロールより優先されてしまう：

```bash
# CI で設定した環境変数が残っている
export AWS_ACCESS_KEY_ID=CI_KEY

# 別のプロファイルを指定したつもりが...
export AWS_PROFILE=production

./myapp
# → CI_KEY が使われる（production プロファイルは無視）
```

**対策:**
- 環境変数のクリーンアップを意識
- `env | grep AWS` で確認習慣
- コンテナ環境では明示的に必要な環境変数だけ設定

**「AssumeRole が動かない」問題**

Credential Chain で credentials は取得できても、config の AssumeRole 設定が読まれないケース：

```bash
# AWS_SDK_LOAD_CONFIG が設定されていない（Go v1）
# → AssumeRole 設定が無視される
```

**解決:**
```bash
export AWS_SDK_LOAD_CONFIG=true
```

**SDK バージョンによる違い**

- AWS SDK for Go v1: `AWS_SDK_LOAD_CONFIG` が必要
- AWS SDK for Go v2: デフォルトで config を読む
- AWS SDK for Python (boto3): デフォルトで config を読む

**セキュリティのベストプラクティス**

Credential Chain の理解は、最小権限原則の実装にも関わる：

- ローカル開発: 限定的な権限のキー
- 本番環境: Instance/Task Role で必要最小限の権限
- 環境変数でのキー渡し: 暗号化・短命化

## 関連ノート（Links）

- [[20251220074112-aws-sdk-load-config-role|AWS_SDK_LOAD_CONFIGの役割と効果]] - config ファイル読み込みの制御
- [[20251220074800-grafana-aws-authentication-sigv4|Grafana AWS認証とSigV4設定]] - Grafana での Credential Chain 活用例

## To-Do / 次に考えること

- [ ] 自社アプリケーションの認証フローを Credential Chain の視点で整理
- [ ] 環境ごとの想定される認証方法を文書化
- [ ] CI/CD での環境変数設定を確認（不要なものが残っていないか）
- [ ] トラブルシューティング用の診断スクリプトを作成
- [ ] AWS SDK のバージョンと Credential Chain の挙動の違いを確認
