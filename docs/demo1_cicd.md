# 【WIP】CICDとIaC化デモ作業ガイド

## 概要
本デモでは、Kong KonnectのData Plane（DP）を「CICD（継続的インテグレーション／継続的デリバリー）」と「Infrastructure as Code（IaC）」の力で、いかに再現性高く・迅速に・安全にデプロイできるかを体験していただきます。

CICDとIaCを活用することで、
- 手作業による属人化やミスを減らす
- デプロイフローの自動化・標準化
- 本番環境に近いテスト環境をすぐ用意できる

といった現代的な運用メリットを最大化できます。

## 得られること
- どの環境でも同じ品質・設定のKong Gatewayをすぐに再現できる
- 人手作業に頼らないことでヒューマンエラーを防ぎ、迅速な運用・障害対応が可能

## 前提条件
- Kong Konnectアカウント、k8s環境
- GitHubリポジトリ/Secrets設定

---

## デモの流れ
| ステップ | 内容                                       | ポイント                 |
| :--- | :--------------------------------------- | :------------------- |
| 1    | Kong Konnectにログインし、デプロイ操作を確認 | クラウド型API管理の統合運用による業務効率化・標準化およびガバナンス強化を実現できること |
| 2    | GitHubでゴールデンイメージ（Dockerfile等）やIaC資材を確認   | インフラ・API資産のコード化による可視化と、再現性・保守性の向上が図れること     |
| 3    | GitHub Actionsでゴールデンイメージを自動ビルド＆脆弱性スキャン   | 継続的な品質担保とセキュリティ強化を実現し、手作業によるミスやばらつきを排除できること |
| 4    | IaCでData Planeノードを自動デプロイ                 | デプロイ作業の自動化により、迅速な構築・再現性の高いインフラ運用が可能となること     |
| 5    | デプロイされたKong Gatewayの動作を確認                | 結果の可視化と構成管理の一元化       |


## 各ステップの詳細手順

### 1. 標準化イメージ（ゴールデンイメージ）の運用確認
1. Kong Konnect管理画面にログイン
   
   ブラウザで Kong Konnect にアクセスし、ご自身のアカウントでログインしてください。
2. 手動によるData Planeノードのデプロイ

   管理画面上で標準イメージの選択やバージョン指定、デプロイ先クラスタの指定など、個別のオペレーションを都度実施することでData Planeを構築します。
   > この手法は操作の柔軟性が高い一方で、作業者ごとの手順差異やヒューマンエラーが生じやすくなります。再現性・標準化・自動化の観点では、後述のCICD/IaCによるデプロイ方式が推奨されます。

### 2.  資材（コード/構成ファイル）確認と環境変数の設定
1. GitHubリポジトリの取得
```
git clone https://github.com/enokidak/kong-bootcamp.git
```

2. ゴールデンイメージ（Dockerfile）の内容確認
```
cat kong-gateway/Dockerfile
# 必要なプラグインや設定が反映されているかご確認ください
```

3. Workflow（CICD定義ファイル）の内容確認
```
cat .github/workflows/build-kong-image.yaml
cat .github/workflows/deploy_dp.yaml
```

4. 環境変数の設定
```
# Azure関連
AKS_RESOURCE_GROUP=<your-azure-resource-group>
AKS_CLUSTER_NAME=<your-aks-cluster>
az ad sp create-for-rbac --name "kongtraining-enk-app" --role contributor \
  --scopes /subscriptions/73ef49b8-168e-4e6e-8e23-a5fed757e4f5/resourceGroups/rg-kong-training \
  --json-auth
AZURE_CREDENTIALS=<your-azure-credentials>

gh variable set AKS_RESOURCE_GROUP --body $AKS_RESOURCE_GROUP --repo $GIT_REPO
gh variable set AKS_CLUSTER_NAME --body $AKS_CLUSTER_NAME --repo $GIT_REPO
gh secret set AZURE_CREDENTIALS --body $AZURE_CREDENTIALS


# Kong関連
KONNECT_TOKEN=<your-konnect-pat>
CP_NAME=default

gh secret set KONNECT_TOKEN --body $KONNECT_TOKEN --repo $GIT_REPO
gh variable set CP_NAME --body $CP_NAME --repo $GIT_REPO


# Github関連
GH_TOKEN=<your-pat>
KONG_REPO=ghcr.io/enokidak/kong-bootcamp/kong-gateway

gh secret set GH_TOKEN --body $GH_TOKEN
gh variable set KONG_REPO --body $KONG_REPO --repo $GIT_REPO
```

### 3. GitHub Actionsnによる、ゴールデンイメージの自動ビルドと脆弱性スキャン
1. GitHub Actions画面へアクセス
   リポジトリの [Actions] タブから「Build and Push Kong Image」ワークフローを開きます。
2. image_tagを指定して実行（例: 3.10.0.5）
   画面右上「Run workflow」をクリックし、タグを入力してビルドを実行します。
3. 進捗・ビルド/スキャン成功を確認
   ワークフロー画面で各ステップが成功しているかご確認ください。
4. ビルド済みイメージの確認

### 4. IaCによる、Data Planeの自動デプロイ
1. GitHub [Actions] → [Deploy Kong Data Plane] を開く
2. `image_tag`に上記で作成したタグを指定し実行
3. 完了後、kubectlやCloudコンソールでPodが立ち上がっていることを確認
