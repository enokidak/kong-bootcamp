# kong-bootcamp

## 1. このリポジトリについて
本リポジトリは、Kong API GatewayおよびKong Konnectの導入・運用を学び、実践できるブートキャンプ用の教材・構成サンプル・自動化資材をまとめたものです。
API管理、CI/CD、APIOps、監査・可観測性など、エンタープライズAPI運用の主要トピックが体系的に体験できます。

---

## 2. 前提条件
- DockerおよびDocker Compose（ローカル実行の場合）
- GitHubアカウント
- AWSアカウント（EKSやマネージドサービス利用時）
- 基本的なKubernetesの知識と`kubectl`コマンド
- Kong Konnectアカウント
- 必要なCLIツール
    - Helm
    - Github CLI
    - Inso CLI
    - decK CLI

※詳細なセットアップや追加ツールについては、各サブディレクトリ内の`README.md`や`docs/`ディレクトリをご確認ください。

---

## 3. 使い方

### クローン

```sh
git clone https://github.com/enokidak/kong-bootcamp.git
cd kong-bootcamp