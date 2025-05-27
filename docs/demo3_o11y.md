# 【WIP】Kong Gateway 可観測性デモ作業ガイド（Prometheus・Grafana・監査ログ連携）

## 概要
本ガイドでは、Kong Gatewayの可観測性をPrometheus/Grafanaで可視化し、加えてKonnectの監査ログ連携までを一連で体験します。
本作業を通じて、「APIゲートウェイの運用状況が“即時に可視化”」「いつ・誰が・何を操作したかも追跡できる」――その運用力を体感していただけます。

## 得られること
- APIリクエストやサービス動向をGrafanaダッシュボードで即可視化
- Prometheusにより細かいメトリクス/可観測性を標準取得
- Kong Konnectの**監査ログ（Audit Log）**で運用監査・セキュリティも強化

## 前提条件
- Kong Konnectアカウント、k8s環境
- Helm, kubectl

---

## デモの流れ
| ステップ | 内容                                           | ポイント                   |
| :--- | :------------------------------------------- | :--------------------- |
| 1    | 監視基盤（cert-manager、Prometheus、Grafana）の準備     | クラウドネイティブな可観測性の実現      |
| 2    | Kong Gateway Data Planeのメトリクス対応構築            | statusAPI/Prometheus連携 |
| 3    | ServiceMonitor/StatusAPI/Prometheus Plugin設定 | 本番さながらのAPI詳細監視         |
| 4    | メトリクスの可視化＆動作確認（Prometheus/Grafana）           | 実際に“見える化”されていることの確認    |
| 5    | Kong Konnect監査ログの確認                          | “いつ誰が何をしたか”も即追跡できる安心感  |



## 各ステップの詳細手順

### 1. 監視基盤の準備（cert-manager、Prometheus、Grafana）
cert-managerインストール（証明書管理用）

```
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.14.4/cert-manager.yaml
# デプロイ状況確認
kubectl get pods -n cert-manager
```

Prometheus & Grafanaのインストール（Helm利用）

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Prometheus Stack（Prometheus + Grafana + いろいろ一式）
helm install prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace

# 動作確認
kubectl get pods -n monitoring
```

### 2. Kong Gateway Data Planeのメトリクス対応構築
a. values.yamlでメトリクス連携設定
values.yaml の該当箇所に以下の設定を追加します（serviceMonitor.enabledやStatusAPI等）：
```
yaml
コピーする
編集する
serviceMonitor:
  enabled: true
  labels:
    release: prometheus-stack

status:
  enabled: true
  http:
    enabled: true
    containerPort: 8100
```

b. HelmでKong Gatewayをデプロイ/アップグレード

```
helm repo add kong https://charts.konghq.com
helm repo update
helm upgrade -i -n kong my-kong kong/kong -f ./values.yaml --debug --wait
```

### 3. メトリクス取得のためのServiceを作成
Status API用にKong PodをServiceで公開します

```
kubectl expose deploy my-kong-kong --port 8100 --name my-kong-kong-status --type ClusterIP -n kong -l enable-metrics="true"
```
※ enable-metrics="true" ラベルがprometheusで拾われます。

### 4. Prometheus/Grafanaによる動作確認
a. Kong経由でAPIリクエスト
Kong上に適当なService・Route作成、PrometheusプラグインをRouteに適用
（作成コマンド例は割愛・GUIやKongKonnect/decKでOK）

- Service例：httpbin-service → https://httpbin.org
- Route例：/httpbin
- Plugin：Prometheus Plugin（Routeに必ずON、すべてのメトリクス有効）

API疎通テスト

```
watch curl http://<KONG-DP-EXTERNAL-URL>/httpbin/anything
```

Prometheusでメトリクス確認

PrometheusのWebUIにアクセス
```
kubectl port-forward svc/prometheus-stack-kube-prometheus-prometheus 9090:9090 -n monitoring
open http://localhost:9090
```

例:
```
kong_http_requests_total{code="200", ..., exported_service="httpbin-service", ...}
```
Grafanaでダッシュボード可視化
- GrafanaのWebUIにアクセス
```
kubectl port-forward svc/prometheus-stack-grafana 3000:80 -n monitoring
open http://localhost:3000
```
- デフォルトユーザ: admin / パスワード: prom-operator（初期値）
- Prometheusをデータソースにし、kong_系メトリクスが見えることを確認

### 5. Kong Konnect 監査ログ（Audit Log）の取得
1. Kong Konnect 管理画面へログイン
https://cloud.konghq.com/ からアクセス

2. [Audit Logs] メニューを開く
APIの作成/変更/削除、設定変更、ユーザ操作等がすべて監査ログとして記録されていることを確認
※（参考: Kong公式ドキュメントより）

3. 用途例
いつ・誰が・どのAPIや設定を操作したかをセキュリティ/運用監査に活用可能


## 本ガイドの意義・ポイント
- APIゲートウェイの可観測性が即時可視化＆分析
- Prometheus/Grafanaで標準監視基盤とシームレス統合
- 監査ログで運用・セキュリティ・ガバナンスも一括担保
- すべてIaC・自動化で“誰がやっても再現”できる構成

>ご質問・ご相談はお気軽にどうぞ
>各コマンドやリソース名はお客様環境に応じて調整してください