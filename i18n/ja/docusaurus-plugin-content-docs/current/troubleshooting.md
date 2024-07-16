# トラブルシューティング

このセクションには、Fleetのトラブルシューティングのためのコマンドとヒントが含まれています。

## **どうやって...**


### `fleet-controller`のログを取得するには？

`fleet-controller`がデプロイされているローカル管理クラスタで、以下のコマンドを実行します。特定の`fleet-controller`ポッド名を入力してください。

```
$ kubectl logs -l app=fleet-controller -n cattle-fleet-system
```

### `fleet-agent`のログを取得するには？

各ダウンストリームクラスタに移動し、以下のコマンドを実行します。特定の`fleet-agent`ポッド名を入力してください。

```
# ダウンストリームクラスタ
$ kubectl logs -l app=fleet-agent -n cattle-fleet-system
# ローカルクラスタ
$ kubectl logs -l app=fleet-agent -n cattle-local-fleet-system
```

### `GitRepos`と`Bundles`の詳細なエラーログを取得するには？

通常、エラーはRancher UIに表示されます。しかし、エラーに関する情報が十分でない場合は、以下のいずれかを試してさらに調査できます。

- バンドルに関する詳細情報を得るには、`bundle`をクリックし、YAMLモードを有効にします。
- GitRepoに関する詳細情報を得るには、`GitRepo`をクリックし、画面右上の`View Yaml`をクリックします。YAMLを表示した後、`status.conditions`を確認します。ここに詳細なエラーメッセージが表示されるはずです。
- 同期エラーについては`fleet-controller`を確認します。
- バンドルのデプロイ時に問題が発生した場合は、ダウンストリームクラスタの`fleet-agent`ログを確認します。

### `GitRepos`と`Bundles`の詳細なステータスを取得するには？

デバッグやバグレポートには、リソースのステータスフィールドの生のJSONが最も役立ちます。これはRancher UIまたは`kubectl`を通じてアクセスできます。

```
kubectl get bundle -n fleet-local fleet-agent-local -o=jsonpath={.status}
kubectl get gitrepo -n fleet-default gitrepo-name -o=jsonpath={.status}
```

### `Kustomize`でチャートレンダリングエラーを確認するには？

[`fleet-controller`のログ](./troubleshooting.md#fetch-the-log-from-fleet-controller)と[`fleet-agent`のログ](./troubleshooting.md#fetch-the-log-from-the-fleet-agent)を確認します。

### `GitRepo`の監視やチェックアウト、または`fleet.yaml`内のダウンロードされたHelmリポジトリに関するエラーを確認するには？

以下のコマンドを使用して、特定の`gitjob`ポッド名を入力して`gitjob-controller`のログを確認します。

```
$ kubectl logs -f $gitjob-pod-name -n cattle-fleet-system
```

ポッドには通常、`rancher/tekton-utils`という名前のイメージがあり、`gitRepo`名がプレフィックスとして付いています。ローカル管理クラスタでこれらのKubernetesジョブポッドのログを確認します。特定の`gitRepoName`ポッド名とネームスペースを入力してください。

```
$ kubectl logs -f $gitRepoName-pod-name -n namespace
```

### `fleet-controller`のステータスを確認するには？

以下のコマンドを実行して、`fleet-controller`ポッドのステータスを確認できます。

```bash
kubectl -n cattle-fleet-system logs -l app=fleet-controller
kubectl -n cattle-fleet-system get pods -l app=fleet-controller
```

```bash
NAME                                READY   STATUS    RESTARTS   AGE
fleet-controller-64f49d756b-n57wq   1/1     Running   0          3m21s
```

### `fleet-controller`と`fleet-agent`のデバッグログを有効にするには？

Rancher v2.6.3 (Fleet v0.3.8) から、デバッグログを有効にする機能が追加されました。

- **ダッシュボード**に移動し、左のナビゲーションメニューで**ローカルクラスタ**をクリックします。
- **アプリ & マーケットプレイス**を選択し、ドロップダウンから**インストール済みアプリ**を選択します。
- そこから、Fleetチャートを`debug=true`の値でアップグレードします。必要に応じて`debugLevel=5`も設定できます。

## **その他のFleet問題に対する追加の解決策**

### CRDの命名規則

1. `clusters`や`gitrepos`のようなCRD用語については、完全なCRD名を参照する必要があります。例えば、クラスタCRDの完全な名前は`cluster.fleet.cattle.io`、gitrepo CRDの完全な名前は`gitrepo.fleet.cattle.io`です。

1. `GitRepo`から作成される`Bundles`は、`GitRepo`が作成された同じワークスペース/ネームスペース内で`$gitrepoName-$path`のパターンに従います。`$path`は、gitリポジトリ内の`bundle`（`fleet.yaml`）を含むディレクトリのパスです。

1. `bundle`から作成される`BundleDeployments`は、`$bundleName-$clusterName`のパターンに従い、ネームスペース`clusters-$workspace-$cluster-$generateHash`に配置されます。`$clusterName`は、バンドルがデプロイされるクラスタの名前です。

### GithubでのHTTPシークレット

プライベートgitリポジトリでFleetをテストする際、GithubではHTTPシークレットがサポートされなくなったことに気付くでしょう。この問題を回避するために、以下の手順に従います。

1. Githubで[個人アクセストークン](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)を作成します。
1. Rancherで、Githubのユーザー名を使用してHTTP[シークレット](https://rancher.com/docs/rancher/v2.6/en/k8s-in-rancher/secrets/)を作成します。
1. トークンをシークレットとして使用します。

### Fleetが不正な応答コード: 403で失敗する

以下のエラーがGitJobに表示される場合、問題はFleetが指定したHelmリポジトリにアクセスできないことが原因かもしれません。

```
time="2021-11-04T09:21:24Z" level=fatal msg="bad response code: 403"
```

以下の手順を実行して評価します。

- リポジトリが開発マシンからアクセス可能であり、Helmチャートを正常にダウンロードできることを確認します。
- gitリポジトリの認証情報が有効であることを確認します。

### Helmチャートリポジトリ: 不明な認証機関によって署名された証明書

以下のエラーがGitJobに表示される場合、誤った証明書チェーンが追加された可能性があります。

```
time="2021-11-11T05:55:08Z" level=fatal msg="Get \"https://helm.intra/virtual-helm/index.yaml\": x509: certificate signed by unknown authority"
```

以下のコマンドで証明書を確認してください。

```bash
context=playground-local
kubectl get secret -n fleet-default helm-repo -o jsonpath="{['data']['cacerts']}" --context $context | base64 -d | openssl x509 -text -noout
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            7a:1e:df:79:5f:b0:e0:be:49:de:11:5e:d9:9c:a9:71
        Signature Algorithm: sha512WithRSAEncryption
        Issuer: C = CH, O = MY COMPANY, CN = NOP Root CA G3
...

```
### Fleetデプロイメントが修正済み状態で停止する

バンドルをFleetにデプロイする際、一部のコンポーネントが変更され、Fleet環境で「修正済み」フラグが立ちます。

`fleet.yaml`でHelmインストールによって生成されたリソースとクラスタ内のリソースの違いを無視するために、`diff.comparePatches`を`fleet.yaml`に追加します。以下の例を参照してください。

```yaml
defaultNamespace: <namespace name>
helm:
  releaseName: <release name>
  repo: <repo name>
  chart: <chart name>
diff:
  comparePatches:
  - apiVersion: apps/v1
    kind: Deployment
    operations:
    - {"op":"remove", "path":"/spec/template/spec/hostNetwork"}
    - {"op":"remove", "path":"/spec/template/spec/nodeSelector"}
    jsonPointers: # jsonPointersは特定のjsonパスでの差分を無視するために使用されます
    - "/spec/template/spec/priorityClassName"
    - "/spec/template/spec/tolerations"
```

どの操作を削除するかを決定するために、ターゲットクラスタの`fleet-agent`のログを確認します。以下のようなエントリが表示されるはずです。

```text
level=error msg="bundle monitoring-monitoring: deployment.apps monitoring/monitoring-monitoring-kube-state-metrics modified {\"spec\":{\"template\":{\"spec\":{\"hostNetwork\":false}}}}"
```

上記のログに基づいて、以下のエントリを追加して操作を削除します。

```json
{"op":"remove", "path":"/spec/template/spec/hostNetwork"}
```

### `GitRepo`または`Bundle`が修正済み状態で停止する

**修正済み**とは、実際の状態と望ましい状態（gitリポジトリにある真実のソース）との間に不一致があることを意味します。

1. 詳細については、[バンドル差分のドキュメント](./bundle-diffs.md)を確認してください。

1. `gitrepo`を強制的に更新して手動で再同期を行うこともできます。左のナビゲーションバーで**GitRepo**を選択し、**強制更新**を選択します。

### バンドルに修正済み状態のHorizontal Pod Autoscaler (HPA)がある

HPAを含むバンドルの場合、バンドルには通常`ReplicaSet`と異なるフィールドが含まれているため、期待される状態は`修正済み`です。

`GitRepo`または`Bundle`が修正済み状態で停止するに従って、このフィールドを無視するパッチを`fleet.yaml`に定義する必要があります。

以下は、ネームスペース`default`のデプロイメント`nginx`のためのパッチの例です。

```yaml
diff:
  comparePatches:
  - apiVersion: apps/v1
    kind: Deployment
    name: nginx
    namespace: default
    operations:
    - {"op": "remove", "path": "/spec/replicas"}
```

### クラスタが利用不可、または`WaitCheckIn`状態の場合はどうするか？

クラスタを再インポートし、登録プロセスを再起動する必要があります。左のナビゲーションバーで**Cluster**を選択し、**強制更新**を選択します。

:::注意

__Rancher v2.5のWaitCheckInステータス__:
クラスタは`WaitCheckIn`ステータスを表示します。これは、`fleet-controller`がRancherサービスIPを使用してFleetと通信しようとしているためです。しかし、Fleetはプロキシを介さずにKubernetesサービスDNSを使用してRancherと直接通信する必要があります。詳細は[Rancherドキュメント](https://rancher.com/docs/rancher/v2.5/en/installation/other-installation-methods/behind-proxy/install-rancher/#install-rancher)を参照してください。

:::

### `gzip: invalid header`でGitRepoがエラーを報告する場合

以下のようなエラーが表示される場合...

```sh
Error opening a gzip reader for /tmp/getter154967024/archive: gzip: invalid header
```

...Helmチャートの内容が正しくありません。チャートを手動でローカルマシンにダウンロードし、内容を確認してください。

### エージェントが登録されなくなった場合

特定のクラスタのエージェントを再デプロイするには、`redeployAgentGeneration`を設定します。

```sh
kubectl patch clusters.fleet.cattle.io -n fleet-local local --type=json -p '[{"op": "add", "path": "/spec/redeployAgentGeneration", "value": -1}]'
```
### ローカルクラスターをFleetのデフォルトクラスターのワークスペースに移行しますか？

ユーザーは新しいワークスペースを作成し、クラスターをワークスペース間で移動することができます。
現在、ローカルクラスターを `fleet-local` から他のワークスペースに移動することはできません。

### バンドルのデプロイに失敗しました: "resource already exists" エラー

デプロイ中に次のエラーメッセージが表示された場合:

```sh
not installed: rendered manifests contain a resource that already
exists. Unable to continue with install: ClusterRole "grafana-clusterrole"
in namespace "" exists and cannot be imported into the current release: invalid
ownership metadata; annotation validation error: key "meta.helm.sh/release-namespace"
must equal "ns-2": current value is "ns-1"
```

このエラーは、同じ `releaseName` を持つHelmリソースがクラスター内に既に存在するために発生します。この問題を解決するには、作成しようとしているリソースの `releaseName` を変更して、競合を避ける必要があります。