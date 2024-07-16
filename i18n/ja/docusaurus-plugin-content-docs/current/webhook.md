# ポーリングの代わりにWebhooksを使用する

デフォルトでは、Fleetはポーリング（デフォルト：15秒ごと）を使用してGitリポジトリからデータを取得します。これは少数のリポジトリ（数十個まで）に対しては便利でうまく機能します。

複数の数十から数百のGitリポジトリを持つインストールや、一般的にレイテンシ（GitへのプッシュとFleetの反応の間の時間）を減らすためには、ポーリングの代わりにWebhooksを設定することをお勧めします。

Fleetは現在、Azure DevOps、GitHub、GitLab、Bitbucket、Bitbucket Server、およびGogsをサポートしています。

### 1. Webhookサービスを設定する。Fleetはgitjobサービスを使用してWebhookリクエストを処理します。gitjobサービスを指すIngressを作成します。

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: webhook-ingress
  namespace: cattle-fleet-system
spec:
  rules:
  - host: your.domain.com
    http:
      paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: gitjob
              port:
                number: 80
```

Rancherや他のサービスと同じホスト名を使用してWebhookを利用したい場合は、以下のYAMLを使用してURL http://your.domain.com/gitjob を設定できます。以下のYAMLはNginx Ingress Controller専用です：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: webhook-ingress
  namespace: cattle-fleet-system
spec:
  rules:
  - host: your.domain.com
    http:
      paths:
        - path: /gitjob(/|$)(.*)
          pathType: ImplementationSpecific
          backend:
            service:
              name: gitjob
              port:
                number: 80
```

:::info

Ingressで[TLS](https://kubernetes.io/docs/concepts/services-networking/ingress/#tls)を設定できます。

:::

### 2. Webhookプロバイダーに移動して、WebhookコールバックURLを設定します。以下はGitHubの例です。

![](/img/webhook.png)

シークレットの設定は任意です。これはWebhookペイロードを検証するために使用されます。ペイロードはデフォルトで信頼されるべきではありません。
Webhookサーバーがインターネットに公開されている場合は、シークレットを設定することをお勧めします。シークレットを設定する場合は、ステップ3に従ってください。

:::note

Webhookライブラリの制限により、application/jsonのみがサポートされています。

:::

:::caution

Webhookを設定した場合、ポーリング間隔は自動的に1時間に調整されます。

:::

### 3. （オプション）Webhookシークレットを設定する。シークレットはWebhookペイロードを検証するためのものです。`cattle-fleet-system`の`gitjob-webhook`というk8sシークレットに設定してください。

| プロバイダー        | K8sシークレットキー     |
|---------------------|-------------------------|
| GitHub              | `github`                |
| GitLab              | `gitlab`                |
| BitBucket           | `bitbucket`             |
| BitBucketServer     | `bitbucket-server`      |
| Gogs                | `gogs`                  |
| Azure DevOps        | `azure-username`        |
| Azure DevOps        | `azure-password`        |

例えば、GitHubシークレットを含むシークレットを作成してWebhookペイロードを検証するには、以下のコマンドを実行します：

```shell
kubectl create secret generic gitjob-webhook -n cattle-fleet-system --from-literal=github=webhooksecretvalue
```

Azure DevOpsの場合：
- Azureで基本認証を有効にする
- 基本認証の資格情報を含むシークレットを作成する
```shell
kubectl create secret generic gitjob-webhook -n cattle-fleet-system --from-literal=azure-username=user --from-literal=azure-password=pass123
```

### 4. Gitプロバイダーに移動して接続をテストします。HTTPレスポンスコードが返されるはずです。