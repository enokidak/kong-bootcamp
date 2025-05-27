# APIOpsデモ作業ガイド

## 概要
本デモでは、BookInfoアプリを例に、APIOpsを用いて、API設計・定義（OpenAPI）からKong Gatewayへのデプロイ、その後のドキュメント管理・自動公開までを一連のCI/CDパイプラインで自動化する流れを体験いただきます。

APIOpsを導入することで…
- API仕様と実装・運用の一貫性の確保
- テストからデプロイ・公開に至るまでのプロセス自動化
- 標準化されたプロセスによる運用品質の向上と属人化の排除

といった、APIの運用管理における信頼性・一貫性・生産性の向上が期待できます。


## 得られること
- OpenAPIの記述・更新がそのままKong GatewayのAPI設定に反映される
- 設計・テスト・差分管理・デプロイ・公開が全自動で再現できる
- API仕様書やAPIドキュメントも自動でKonnect Dev Portalに反映される


## 前提条件
- Kong KonnectアカウントおよびControl Planeの作成、Data Planeをデプロイ済み
- API Product/Versionの作成済み（Konnect側で初期設定）
- GitHubリポジトリ/Secrets/環境変数設定済み
- API定義（OpenAPI Spec api-spec.yaml）、API Productドキュメント（product.md）用意済み
- Kubernetesクラスター利用可
- BookInfoリポジトリ取得済み
- deck CLI導入済み（またはGHA内コンテナで利用）

---

## デモの流れ
| ステップ | 内容                                     | ポイント            |
| :--- | :------------------------------------- | :-------------- |
| 1    | OpenAPI Spec/ドキュメント/CI資材がGitHubで一元管理されていることを確認     | すべてのAPI関連資産が構成管理・自動化されている      |
| 2    | BookInfoアプリをKubernetes上にデプロイ           | 本番運用を想定したAPI環境の迅速な構築を実演  |
| 3    | BookInfoアプリを外部公開（Ingress/Kong連携）       | 実際のAPI公開運用を踏まえた接続イメージを提示     |
| 4    | Kong GatewayへBookInfo設定をデプロイ | バックエンドAPIをKongを利用して一元管理  |
| 5    | APIOps（OpenAPI→Kong自動反映）のCI/CDパイプラインを実行 | API変更が即座かつ一貫性を持って適用されるプロセスを実証 |
| 6    | 管理画面・Dev Portal等でAPI/ドキュメント反映を確認       | 運用結果の可視化および再現性の高さを確認 |



## 各ステップの詳細手順

### 1.  資材（コード/構成ファイル）確認と環境変数の設定
1. GitHubリポジトリの取得
```
git clone https://github.com/enokidak/kong-bootcamp.git
```

2. Workflow（CICD定義ファイル）の内容確認
```
cat .github/workflows/deploy_oas.yaml
cat .github/workflows/upload_doc.yaml
cat .github/workflows/upload_spec.yaml
```

3. 環境変数の設定
```
GIT_REPO=enokidak/kong-bootcamp

# Kong関連
KONNECT_REGION=us
CONTROL_PLANE=default
TAG=bookinfo

gh variable set KONNECT_REGION --body $KONNECT_REGION --repo $GIT_REPO
gh variable set CONTROL_PLANE --body $CONTROL_PLANE --repo $GIT_REPO
gh variable set TAG --body $TAG --repo $GIT_REPO

KONNECT_TOKEN=<your-konnect-pat>
gh secret set KONNECT_TOKEN --body $KONNECT_TOKEN --repo $GIT_REPO
```


### 2. BookInfoアプリのKubernetesデプロイ

1. サンプルアプリ公開用のIngress Controller(Contour)をデプロイ
```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install contour bitnami/contour --namespace projectcontour --create-namespace
```

2. Namespaceの作成

```
kubectl create ns bookinfo
```

3. BookInfoアプリ本体のデプロイ

```
kubectl apply -f bookinfo/platform/kube/bookinfo.yaml -n bookinfo
```

4. Ingressリソース適用（Kong Gatewayと連携）

```
kubectl apply -f bookinfo/productpage-ingress.yaml
```

### 3. BookInfoサービスがKubernetes上で稼働していることを確認
```
kubectl get pods -n bookinfo
kubectl get svc -n bookinfo
kubectl get ingress
```

- すべてのPodがRunningになっていること
- Ingressが作成されていること


### 4. Kong GatewayへBookInfo用設定（Kongリソース）を反映
1. deck用マニフェストの確認
※ deck-bookinfo.yaml などでKong用の設定を用意（Service/Route/Plugin等）

2. deck CLIでKonnectへ反映
```
deck --konnect-addr https://us.api.konghq.com \
  --konnect-control-plane-name default \
  --konnect-token <KONNECT_TOKEN> \
  gateway sync deck-bookinfo.yaml
```
※ <KONNECT_TOKEN>はKong KonnectのAPIキーに置き換えてください


### 5. APIOps（OpenAPI→Kong自動反映）によりKong GatewayへBookInfo設定をデプロイ
1. OpenAPI Specを編集（例: エンドポイント追加/修正）

```
vi bookinfo/openapi.yaml
```

2. GitHubにコミット・プッシュ
```
git add bookinfo/openapi.yaml
git commit -m "Update API Spec for BookInfo"
git push origin main
```

3. GitHub Actions でワークフロー起動・進捗確認

- [Actions]タブ → 「Convert OpenAPI Spec to Kong and Deploy」を確認
- Lint→変換→差分確認→Kong Gatewayへの自動反映が実行されていることを確認

### 6. Dev Portal・API管理画面でAPI/ドキュメント反映を確認
- Kong Konnect の管理画面でAPI定義・エンドポイントが反映されていることを確認
- Dev PortalでAPI仕様やドキュメントが最新化されていることを確認

### （オプション）API疎通テスト
- BookInfoの外部エンドポイントにcurl等でアクセスし疎通
```
curl -i https://<外部公開FQDN>/productpage
```

- 正常レスポンス（200等）が返ることを確認

---

## 補足
- BookInfo向けKong/Ingressリソース（deck-bookinfo.yaml, productpage-ingress.yaml）はサンプルとしてご用意しています。
- OpenAPI→Kong変換～デプロイのパイプライン（APIOps GHA）は、環境に応じてワークフロー定義ファイルを調整してください。

