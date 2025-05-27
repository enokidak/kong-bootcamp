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
| 1    | Kong Konnect管理画面にログインし、標準化イメージの運用イメージを確認 | SaaS運用＋標準化＝現場負担軽減の訴求 |
| 2    | GitHubでゴールデンイメージ（Dockerfile等）やIaC資材を確認   | 全資産がコード化されている安心感     |
| 3    | GitHub Actionsでゴールデンイメージを自動ビルド＆脆弱性スキャン   | “誰がやっても同じ・早い・安全”を見せる |
| 4    | IaCでData Planeノードを自動デプロイ                 | ワンクリック再現性＆自動化の強み     |
| 5    | デプロイされたKong Gatewayの動作を確認                | 結果が見える！再現できる！        |


## 各ステップの詳細手順

### 1. 標準化イメージ（ゴールデンイメージ）の運用確認
1. Kong Konnect管理画面にログイン
   ブラウザで Kong Konnect にアクセスし、ご自身のアカウントでログインしてください。
2. 「ゴールデンイメージ」運用イメージの可視化
   SaaSの統合管理で標準イメージやライフサイクルが一元管理されている点を確認します。
   管理画面上でのバージョン管理や適用状況もご覧いただけます。

### 2.  資材（コード/構成ファイル）確認
1. GitHubリポジトリの取得またはブラウザ参照
```
git clone https://github.com/<ORG>/<REPO>.git
cd <REPO>
```

2. ゴールデンイメージ（Dockerfile）の内容確認
```
cat kong-gateway/Dockerfile
# 必要なプラグインや設定が反映されているかご確認ください
```

3. IaC（インフラ定義ファイル）の内容確認
```
ls -l iac/
cat iac/values.yaml
# KubernetesやHelm、Terraform等の定義内容をご確認ください
```


### 3. GitHub Actionsnによる、ゴールデンイメージの自動ビルドと脆弱性スキャン
1. GitHub Actions画面へアクセス
   リポジトリの [Actions] タブから「Build and Push Kong Image」ワークフローを開きます。
2. image_tagを指定して実行（例: 3.10.0.5）
   画面右上「Run workflow」をクリックし、タグを入力してビルドを実行します。
3. 進捗・ビルド/スキャン成功を確認
   ワークフロー画面で各ステップが成功しているかご確認ください。
4. ビルド済みイメージの確認
```
# ghcr.ioにpushされたことをWebで確認
https://github.com/users/enokidak/packages/container/package/kong-bootcamp%2Fkong-gateway
```

### 4. IaCによる、Data Planeの自動デプロイ
1. GitHub [Actions] → [Deploy Kong Data Plane] を開く
2. `image_tag`に上記で作成したタグを指定し実行
3. 完了後、kubectlやCloudコンソールでPodが立ち上がっていることを確認

### 5. 動作確認
1. サービス作成
2. ルート作成
3.環境内から疎通を試みる。CICDによってデプロイしたKong DPを使用できていることを確認する