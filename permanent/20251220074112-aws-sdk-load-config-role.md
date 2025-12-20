---
id: 2025-12-20-074112
title: AWS_SDK_LOAD_CONFIGの役割と効果
created: 2025-12-20 07:41:12
updated: 2025-12-20 07:41:12
tags:
  - zettel
  - aws
  - sdk
  - configuration
  - environment-variable
source:
  type: concept
  ref: "AWS SDK for Go v1 documentation"
---

## 要約（Summary）

- `AWS_SDK_LOAD_CONFIG` は AWS SDK が `~/.aws/config` を読み込むかを制御する環境変数
- この環境変数を `true` にすると、credentials だけでなく config ファイルの設定も有効になる
- AWS SDK for Go v1 ではデフォルトで無効（`false`）、明示的な有効化が必要
- Region、AssumeRole、プロファイル設定など config 依存の機能が動作可能になる

## 本文（Body）

`AWS_SDK_LOAD_CONFIG` は、AWS SDK（特に Go v1）における「shared config ファイルの読み込み」を制御するスイッチとして機能する環境変数である。この変数の理解は、AWS 認証周りのトラブルシューティングで頻繁に必要になる。

### 背景・問題意識

**AWS SDK の設定ファイル構造**

AWS SDK は2種類の設定ファイルを持つ：
- `~/.aws/credentials` - 認証情報（アクセスキー・シークレット）
- `~/.aws/config` - その他の設定（リージョン、ロール、プロファイル設定）

**デフォルト動作の違い**

SDK によって「config を読むかどうか」のデフォルトが異なる：
- AWS SDK for Go v1: デフォルトで **config を読まない**
- AWS CLI / 他の SDK: 多くはデフォルトで読む

この違いが「ローカルでは動くのにアプリケーションでは動かない」問題を引き起こす。

### アイデア・主張

**`AWS_SDK_LOAD_CONFIG=true` を設定することで、AWS SDK は credentials と config の両方を読み込み、プロファイル・リージョン・AssumeRole など config 依存の機能が有効化される。**

#### 何が起こるか

**設定なし（デフォルト）の場合:**
```bash
# ~/.aws/config の内容
[profile myprofile]
region = ap-northeast-1
role_arn = arn:aws:iam::123456789012:role/MyRole
source_profile = default
```

```go
// Go アプリケーション
sess := session.Must(session.NewSessionWithOptions(session.Options{
    Profile: "myprofile",
}))
// → region が取得できない
// → AssumeRole が動作しない
```

**AWS_SDK_LOAD_CONFIG=true の場合:**
```bash
export AWS_SDK_LOAD_CONFIG=true
```

```go
// 同じコード
sess := session.Must(session.NewSessionWithOptions(session.Options{
    Profile: "myprofile",
}))
// → ~/.aws/config から region が読まれる
// → AssumeRole が正しく動作する
```

#### 具体的に有効になる設定

**1) Region 設定**
```ini
# ~/.aws/config
[profile production]
region = us-east-1
```

`AWS_SDK_LOAD_CONFIG=true` があれば、このリージョン設定が SDK に反映される。

**2) AssumeRole 設定**
```ini
# ~/.aws/config
[profile assume-role-profile]
role_arn = arn:aws:iam::999999999999:role/TargetRole
source_profile = default
```

SDK が自動的に STS AssumeRole を実行する。

**3) カスタム設定ファイルパス**
```bash
export AWS_CONFIG_FILE=/custom/path/to/config
export AWS_SDK_LOAD_CONFIG=true
```

デフォルト以外の場所にある config も読み込まれる。

### 内容を視覚化するMermaid図

```mermaid
graph TB
    subgraph "AWS_SDK_LOAD_CONFIG=false (デフォルト)"
        A1["AWS SDK起動"] --> B1["~/.aws/credentials<br/>のみ読み込み"]
        B1 --> C1["アクセスキー取得"]
        B1 -.-|読まない| D1["~/.aws/config"]
        C1 --> E1["❌ Regionなし<br/>❌ AssumeRoleなし<br/>❌ Profileなし"]
    end
    
    subgraph "AWS_SDK_LOAD_CONFIG=true"
        A2["AWS SDK起動"] --> B2["~/.aws/credentials<br/>読み込み"]
        A2 --> C2["~/.aws/config<br/>読み込み"]
        B2 --> D2["アクセスキー取得"]
        C2 --> E2["Region取得"]
        C2 --> F2["AssumeRole設定"]
        C2 --> G2["Profile設定"]
        
        D2 --> H2["✓ 完全な認証情報"]
        E2 --> H2
        F2 --> H2
        G2 --> H2
    end
    
    H2 --> I["AWS API呼び出し成功"]
    E1 --> J["AWS API呼び出し失敗<br/>（設定不足）"]
    
    style H2 fill:#ccffcc
    style I fill:#ccffcc
    style E1 fill:#ffcccc
    style J fill:#ffcccc
```

### 具体例・ケース

#### ケース1: Grafana で AMP にアクセス

**問題:**
```bash
# Grafana が AMP（Amazon Managed Prometheus）に接続できない
Error: missing region
```

**原因:**
- Grafana のバックエンドが AWS SDK for Go を使用
- `~/.aws/config` に region が書いてあるが読まれていない

**解決:**
```bash
# Grafana 起動時
export AWS_SDK_LOAD_CONFIG=true
export GF_AUTH_SIGV4_AUTH_ENABLED=true
./bin/grafana-server
```

または Docker の場合:
```yaml
# docker-compose.yml
services:
  grafana:
    environment:
      - AWS_SDK_LOAD_CONFIG=true
      - GF_AUTH_SIGV4_AUTH_ENABLED=true
    volumes:
      - ~/.aws:/root/.aws:ro
```

#### ケース2: AssumeRole を使った本番アクセス

**構成:**
```ini
# ~/.aws/credentials
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

# ~/.aws/config
[profile production]
role_arn = arn:aws:iam::123456789012:role/ProductionRole
source_profile = default
region = ap-northeast-1
```

**アプリケーション:**
```go
// main.go
os.Setenv("AWS_SDK_LOAD_CONFIG", "true")
sess := session.Must(session.NewSessionWithOptions(session.Options{
    Profile: "production",
    SharedConfigState: session.SharedConfigEnable,
}))

svc := s3.New(sess)
// → default のキーで ProductionRole を AssumeRole
// → ap-northeast-1 リージョンで S3 操作
```

#### ケース3: 設定がなくても動くケース（比較）

**EC2 インスタンスロール環境:**
```bash
# EC2 上で実行
# → Instance Metadata Service から自動取得
# → AWS_SDK_LOAD_CONFIG 不要
```

**環境変数で直接指定:**
```bash
export AWS_ACCESS_KEY_ID=xxx
export AWS_SECRET_ACCESS_KEY=yyy
export AWS_REGION=us-west-2
# → config ファイル不要
# → AWS_SDK_LOAD_CONFIG 不要
```

ただし、これらの方法では「プロファイルを使い分けたい」「AssumeRole を自動でやりたい」という要求に応えられない。

### 反論・限界・条件

**「環境変数で全部やればいいのでは？」**

確かに、小規模・単一環境なら：
```bash
export AWS_ACCESS_KEY_ID=xxx
export AWS_SECRET_ACCESS_KEY=yyy
export AWS_REGION=us-east-1
```
で済む。

しかし：
- 複数アカウント・複数ロールの使い分けができない
- AssumeRole の自動実行ができない
- 既存の AWS CLI 設定を再利用できない

**「AWS CLI では設定なしで動く」**

AWS CLI は内部的に `AWS_SDK_LOAD_CONFIG=true` 相当の動作をデフォルトでする。そのため CLI では問題ないのに、Go アプリケーションで問題が起きる。

**SDK バージョンによる違い**

- AWS SDK for Go v1: `AWS_SDK_LOAD_CONFIG` が必要
- AWS SDK for Go v2: デフォルトで config を読む（環境変数不要）

移行を検討する場合は v2 の方が直感的。

**セキュリティ考慮**

config ファイルに機密情報を書かないこと：
- ✓ role_arn, region - OK
- ❌ aws_access_key_id, aws_secret_access_key - credentials に分離

## 関連ノート（Links）

- [[20251220074500-aws-sdk-credential-chain|AWS SDK Credential Chainの動作メカニズム]] - SDKがどの順序で認証情報を探すか
- [[20251220074800-grafana-aws-authentication-sigv4|Grafana AWS認証とSigV4設定]] - Grafanaで AWS サービスにアクセスする際の認証設定

## To-Do / 次に考えること

- [ ] 自社アプリケーションで AWS_SDK_LOAD_CONFIG の設定状況を確認
- [ ] Dockerfile/docker-compose.yml に環境変数が設定されているか確認
- [ ] AWS SDK for Go v2 への移行を検討（デフォルトで config を読む）
- [ ] CI/CD 環境での config ファイルマウント方法を整理
