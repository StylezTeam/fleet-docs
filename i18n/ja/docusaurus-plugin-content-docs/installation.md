```markdown
import {versions} from '@site/src/fleetVersions';
import CodeBlock from '@theme/CodeBlock';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# インストール詳細

インストールは、シングルクラスターとマルチクラスターの2つのユースケースに分かれています。
シングルクラスターインストールは、GitOpsを使用して単一のクラスターを管理したい場合に適しています。
この場合、集中管理クラスターは必要ありません。マルチクラスターのユースケースでは、
集中管理クラスターをセットアップし、そこにクラスターを登録します。

Fleetを学習するだけなら、シングルクラスターインストールが推奨されます。
その後、シングルクラスターからマルチクラスターのセットアップに移行できます。

![](/img/single-cluster.png)

シングルクラスターはデフォルトのインストールです。同じクラスターがFleetマネージャーとFleetエージェントの両方を実行します。
クラスターはGitサーバーと通信して、このローカルクラスターにリソースをデプロイします。
これは最もシンプルなセットアップで、開発/テストや小規模なセットアップに非常に便利です。
このユースケースは、プロダクションの有効なユースケースとしてサポートされています。

## 前提条件

<Tabs>
  <TabItem value="helm" label="Helm 3" default>
  FleetはHelmチャートとして配布されています。Helm 3はCLIであり、サーバーサイドコンポーネントはなく、非常に簡単です。
  Helm 3 CLIをインストールするには、<a href="https://helm.sh/docs/intro/install">公式インストール手順</a>に従ってください。
  </TabItem>
  <TabItem value="kubernetes" label="Kubernetes" default>
  FleetはKubernetesクラスター上で動作するコントローラーなので、既存のクラスターが必要です。
  シングルクラスターのユースケースでは、GitOpsで管理する予定のクラスターにFleetをインストールします。
  Kubernetesコミュニティがサポートする任意のバージョンのKubernetesが動作します。実際には{versions.next.kubernetes}以上のバージョンが必要です。
  </TabItem>
</Tabs>

## デフォルトインストール

以下の2つのHelmチャートをインストールします。

<Tabs>
<TabItem value="install" label="インストール" default>

:::caution RancherのFleet
RancherにはFleet用の別のHelmチャートがあり、異なるリポジトリを使用します。
:::

まず、FleetのHelmリポジトリを追加します。
<CodeBlock language="bash">
{`helm repo add fleet https://rancher.github.io/fleet-helm-charts/`}
</CodeBlock>

次に、FleetのCustomResourcesDefintionsをインストールします。
<CodeBlock language="bash">
{`helm -n cattle-fleet-system install --create-namespace --wait fleet-crd \\
    fleet/fleet-crd`}
</CodeBlock>

最後に、Fleetコントローラーをインストールします。
<CodeBlock language="bash">
{`helm -n cattle-fleet-system install --create-namespace --wait fleet \\
    fleet/fleet`}
</CodeBlock>
</TabItem>
<TabItem value="verify" label="確認">

Fleetはシングルクラスターで使用できるようになっているはずです。以下のコマンドを実行して、Fleetコントローラーポッドのステータスを確認できます。

```bash
kubectl -n cattle-fleet-system logs -l app=fleet-controller
kubectl -n cattle-fleet-system get pods -l app=fleet-controller
```

```
NAME                                READY   STATUS    RESTARTS   AGE
fleet-controller-64f49d756b-n57wq   1/1     Running   0          3m21s
```
</TabItem>
</Tabs>

これで、`fleet-local`ネームスペースにいくつかのGitリポジトリを[登録](./gitrepo-add.md)して、Kubernetesリソースのデプロイを開始できます。

## マルチコントローラーインストール: シャーディング

### デプロイメント

バージョン0.10以降、Fleetは静的シャーディングをサポートしています。Fleetコントローラーチャートは`--set shards={<カンマ区切りのシャードID>}`でインストールでき、以下の結果が得られます:
* 指定されたユニークなシャードIDの数だけFleetコントローラーデプロイメントが作成されます。
* 通常の非シャーディングFleetコントローラーポッドも作成されます。このポッドはエージェント管理とクリーンアップコンテナを含む唯一のポッドです。

例えば:
```bash
$ helm -n cattle-fleet-system install --create-namespace --wait --set shards="{foo,bar,baz}" \
fleet fleet/fleet

$ kubectl -n cattle-fleet-system get pods -l app=fleet-controller
NAME                                          READY   STATUS    RESTARTS      AGE
fleet-controller-78c74fdb85-b6q64             3/3     Running   0             77s
fleet-controller-shard-bar-777d888865-w2dks   1/1     Running   0             77s
fleet-controller-shard-baz-6595bd9cb9-27whg   1/1     Running   0             77s
fleet-controller-shard-foo-85d49b446f-pzxkw   1/1     Running   0             77s

$ kubectl -n cattle-fleet-system get pods -l app=fleet-controller \
-o=jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.fleet\.cattle\.io/shard-id}{"\n"}{end}'
fleet-controller-78c74fdb85-b6q64
fleet-controller-shard-bar-777d888865-w2dks     bar
fleet-controller-shard-baz-6595bd9cb9-27whg     baz
fleet-controller-shard-foo-85d49b446f-pzxkw     foo
```

### 仕組み

シャーディングが設定されると、各Fleetコントローラーは自分のシャードIDを持つリソースを処理します。非シャーディングコントローラーも同様で、シャードIDが設定されていないため、すべての非シャーディングリソースを処理します。

特定のシャード用にGitRepoをデプロイするには、ラベル`fleet.cattle.io/shard-ref`に希望するシャードIDを値として追加します。
以下はその例です:
```bash
$ kubectl apply -n fleet-local -f - <<EOF
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: sharding-test
  labels:
    fleet.cattle.io/shard-ref: foo
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - single-cluster/helm
EOF
```

FleetコントローラーがデプロイされているラベルID（上記の例では`foo`）を持つGitRepoは、そのコントローラーによって処理されます。

一方、未知のラベルID（上記の例では`boo`）を持つGitRepoは、どのFleetコントローラーによっても処理されないため、GitRepo自体以外のリソースは作成されません。

サポートされているシャードIDの追加や削除には、現在のところ、新しいシャードIDセットでFleetを再デプロイする必要があります。

## マルチクラスターの設定

:::caution
Rancherのダウンストリームクラスターは自動的にFleetに登録されます。ユーザーはRancherの`Continuous Delivery`でFleetにアクセスできます。

以下に説明するマルチクラスターインストールは、スタンドアロンのFleetで  **のみ** カバーされており、Rancher QAによってテストされていません。
:::


:::info
セットアップはシングルクラスターと同じです。
Fleetマネージャーをインストールした後、リモートのダウンストリームクラスターをFleetマネージャーに登録する必要があります。

ただし、ダウンストリームクラスターの[マネージャーによる登録](./cluster-registration.md#manager-initiated)を許可するために、いくつかの追加設定が必要です。APIサーバーURLとCAがない場合、ダウンストリームクラスターの[エージェントによる登録](./cluster-registration.md#agent-initiated)のみが可能です。
:::

### APIサーバーURLとCA証明書

Fleet管理インストールが正しく機能するためには、正しいAPIサーバーURLとCA証明書が適切に設定されていることが重要です。FleetエージェントはKubernetes APIサーバーURLに通信します。つまり、Kubernetes APIサーバーはダウンストリームクラスターからアクセス可能でなければなりません。また、APIサーバーのCA証明書を取得する必要があります。この情報を取得する最も簡単な方法は、通常、kubeconfigファイル（`$HOME/.kube/config`）からです。`server`、`certificate-authority-data`、または`certificate-authority`フィールドにこれらの値が含まれています。

```yaml title="$HOME/.kube/config"
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTi...
    server: https://example.com:6443
```

#### CA証明書の抽出

`certificate-authority-data`フィールドはbase64エンコードされており、ファイルに保存する前にデコードする必要があります。これは、base64エンコードされた内容をファイルに保存し、その後に以下のコマンドを実行することで行えます。

```shell
base64 -d encoded-file > ca.pem
```

次に、kubeconfigからCA証明書を取得します。

<Tabs>
<TabItem value="extractca" label="最初に抽出">
`jq`と`base64`が利用可能な場合、このワンライナーで`KUBECONFIG`からすべてのCA証明書を取得し、`ca.pem`というファイルに保存します。

```shell
kubectl config view -o json --raw  | jq -r '.clusters[].cluster["certificate-authority-data"]' | base64 -d > ca.pem
```
</TabItem>
<TabItem value="extractcas" label="複数のエントリ">
また、マルチクラスターセットアップの場合、以下のコマンドを使用できます。

```shell
# KUBECONFIGに従ってクラスターの名前をCLUSTERNAMEに置き換えます
kubectl config view -o json --raw  | jq -r '.clusters[] | select(.name=="CLUSTERNAME").cluster["certificate-authority-data"]' | base64 -d > ca.pem
```
</TabItem>
</Tabs>


#### APIサーバーの抽出

マルチクラスターセットアップの場合、以下のコマンドを使用できます。

```shell
# KUBECONFIGに従ってクラスターの名前をCLUSTERNAMEに置き換えます
API_SERVER_URL=$(kubectl config view -o json --raw  | jq -r '.clusters[] | select(.name=="CLUSTER").cluster["server"]')
# APIサーバーがよく知られたCAによって署名されている場合は空のままにします
API_SERVER_CA="ca.pem"
```

#### 検証

まず、サーバーURLが正しいことを確認します。

```shell
curl -fLk "$API_SERVER_URL/version"
```

このコマンドの出力は、Kubernetesサーバーのバージョンを示すJSONか、`401 Unauthorized`エラーである必要があります。これらの結果が得られない場合は、URLが正しいことを確認してください。KubernetesのAPIサーバーポートは通常6443です。

次に、以下のコマンドを実行してCA証明書が正しいことを確認します。APIサーバーがよく知られたCAによって署名されている場合は、コマンドの`--cacert "$API_SERVER_CA"`部分を省略します。

```shell
curl -fL --cacert "$API_SERVER_CA" "$API_SERVER_URL/version"
```

有効なJSONレスポンスまたは`401 Unauthorized`が得られた場合、成功です。Unauthorizedエラーは、curlコマンドが適切な認証情報を設定していないためですが、TLS接続が機能し、`ca.pem`がこのURLに対して正しいことを確認できます。`SSL certificate problem`エラーが発生した場合、`ca.pem`が正しくありません。`$API_SERVER_CA`ファイルの内容は以下のようになります:

```pem title="ca.pem"
-----BEGIN CERTIFICATE-----
MIIBVjCB/qADAgECAgEAMAoGCCqGSM49BAMCMCMxITAfBgNVBAMMGGszcy1zZXJ2
ZXItY2FAMTU5ODM5MDQ0NzAeFw0yMDA4MjUyMTIwNDdaFw0zMDA4MjMyMTIwNDda
MCMxITAfBgNVBAMMGGszcy1zZXJ2ZXItY2FAMTU5ODM5MDQ0NzBZMBMGByqGSM49
AgEGCCqGSM49AwEHA0IABDXlQNkXnwUPdbSgGz5Rk6U9ldGFjF6y1YyF36cNGk4E
0lMgNcVVD9gKuUSXEJk8tzHz3ra/+yTwSL5xQeLHBl+jIzAhMA4GA1UdDwEB/wQE
AwICpDAPBgNVHRMBAf8EBTADAQH/MAoGCCqGSM49BAMCA0cAMEQCIFMtZ5gGDoDs
ciRyve+T4xbRNVHES39tjjup/LuN4tAgAiAteeB3jgpTMpZyZcOOHl9gpZ8PgEcN
KDs/pb3fnMTtpA==
-----END CERTIFICATE-----
```

### マルチクラスターのインストール

以下の例では、`KUBECONFIG`からのAPIサーバーURLが`https://example.com:6443`であり、CA証明書が`ca.pem`ファイルにあると仮定します。もしAPIサーバーURLがよく知られたCAによって署名されている場合、以下の`apiServerCA`パラメータを省略するか、空の`ca.pem`ファイルを作成するだけで済みます（例：`touch ca.pem`）。

特定の値で環境を設定します。例：

```shell
API_SERVER_URL="https://example.com:6443"
API_SERVER_CA="ca.pem"
```

APIサーバーURLとAPIサーバーCAパラメータを検証したら、以下の2つのHelmチャートをインストールします。

<Tabs>
<TabItem value="install2" label="インストール" default>
まず、FleetのHelmリポジトリを追加します。
<CodeBlock language="bash">
{`helm repo add fleet https://rancher.github.io/fleet-helm-charts/`}
</CodeBlock>

次に、FleetのCustomResourcesDefintionsをインストールします。
<CodeBlock language="bash">
{`helm -n cattle-fleet-system install --create-namespace --wait \\
    fleet-crd`} {versions.next.fleetCRD}
</CodeBlock>

最後に、Fleetコントローラーをインストールします。
<CodeBlock language="bash">
{`helm -n cattle-fleet-system install --create-namespace --wait \\
    --set apiServerURL="$API_SERVER_URL" \\
    --set-file apiServerCA="$API_SERVER_CA" \\
    fleet`} {versions.next.fleet}
</CodeBlock>
</TabItem>

<TabItem value="verifiy2" label="確認">
Fleetが使用可能であることを確認します。以下のコマンドを実行して、Fleetコントローラーポッドのステータスを確認できます。

```bash
kubectl -n cattle-fleet-system logs -l app=fleet-controller
kubectl -n cattle-fleet-system get pods -l app=fleet-controller
```

```
NAME                                READY   STATUS    RESTARTS   AGE
fleet-controller-64f49d756b-n57wq   1/1     Running   0          3m21s
```
</TabItem>
</Tabs>

この時点で、Fleetマネージャーが準備完了です。これで[Fleetマネージャーにクラスターを登録](./cluster-registration.md)し、[Gitリポジトリを追加](./gitrepo-add.md#create-gitrepo-instance)できます。
```