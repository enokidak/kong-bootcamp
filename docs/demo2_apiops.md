# 【WIP】APIOpsデモ作業ガイド

## 概要
本デモでは、BookInfoアプリを例に、APIOpsを用いて、API設計・定義（OpenAPI）からKong Gatewayへのデプロイ、その後のドキュメント管理・自動公開までを一連のCI/CDパイプラインで自動化する流れを体験いただきます。

APIOpsを導入することで…
- API仕様と実装・管理の“ズレ”を防ぐ
- テスト・差分検証・公開まで一気通貫で自動化
- 属人作業を極小化し、開発・運用の品質を統一
といった現代的なAPI運用メリットを最大化できます。



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
| 1    | OpenAPI Spec/ドキュメント/CI資材がGitHubで一元管理されていることを確認     | 全API資産が**コード化・自動化**       |
| 2    | BookInfoアプリをKubernetes上にデプロイ           | 本番さながらのAPIを即用意  |
| 3    | BookInfoアプリを外部公開（Ingress/Kong連携）       | API運用の実際を体験     |
| 4    | Kong GatewayへBookInfo設定をデプロイ           | API管理自動化の第一歩    |
| 5    | APIOps（OpenAPI→Kong自動反映）CI/CDパイプラインを実行 | “誰がやっても同じ・すぐ反映” |
| 6    | 管理画面・Dev Portal等でAPI/ドキュメント反映を確認       | 結果が**見える化・再現性** |



## 各ステップの詳細手順

### 1.  資材（コード/構成ファイル）確認
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


### 2. BookInfoアプリのKubernetesデプロイ
0. アプリ公開用のContourをデプロイ
Contourのインストール
今回はディストリビューション依存をなくすため、Tanzu Packagesは使わずに普通にOSS版のContourをインストールする。

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install contour bitnami/contour --namespace projectcontour --create-namespace
```
Podが立ち上がってEnvoyのServiceにExternal-IPが割り振られればOK。
```
kubectl get all -n projectcontour
NAME                                   READY   STATUS    RESTARTS   AGE
pod/contour-contour-64c7959cf9-vk684   1/1     Running   0          91s
pod/contour-envoy-b72ld                2/2     Running   0          91s
pod/contour-envoy-bkxvp                2/2     Running   0          91s
pod/contour-envoy-plspq                2/2     Running   0          91s

NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)                      AGE
service/contour         ClusterIP      10.102.173.86    <none>           8001/TCP                     91s
service/contour-envoy   LoadBalancer   10.100.171.238   10.214.154.165   80:32533/TCP,443:31826/TCP   91s

NAME                           DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/contour-envoy   3         3         3       3            3           <none>          91s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/contour-contour   1/1     1            1           91s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/contour-contour-64c7959cf9   1         1         1       92s
```

1. Namespaceの作成

```
kubectl create ns bookinfo
```

2. BookInfoアプリ本体のデプロイ

```
kubectl apply -f bookinfo/platform/kube/bookinfo.yaml -n bookinfo
```

3. Ingressリソース適用（Kong Gatewayと連携）

```
kubectl apply -f bookinfo/productpage-ingress.yaml
```

※ bookinfoディレクトリはGitHub等からcloneしてください。

### 3. BookInfoサービスがKubernetes上で稼働していることを確認
```
kubectl get pods -n bookinfo
kubectl get svc -n bookinfo
kubectl get ingress
```

- すべてのPodがRunningになっていること
- Ingress（Kong）が作成されていること

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

### 5. APIOps（OpenAPI→Kong自動反映）ワークフロー実行
1. OpenAPI Specを編集（例: エンドポイント追加/修正）

```
vi apiops/docs/openapi/api-spec.yaml
```

2. GitHubにコミット・プッシュ
```
git add apiops/docs/openapi/api-spec.yaml
git commit -m "Update API Spec for BookInfo"
git push origin main
```

3. GitHub Actions でワークフロー起動・進捗確認

- [Actions]タブ → 「Convert OpenAPI Spec to Kong and Deploy」を確認
- Lint→変換→差分確認→Kong Gatewayへの自動反映

※ コマンド実行例は前述APIOps手順と同様（deck, inso, curl など）

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

