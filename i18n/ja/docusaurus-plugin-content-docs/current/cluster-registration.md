import {versions} from '@site/src/fleetVersions';
import CodeBlock from '@theme/CodeBlock';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# ダウンストリームクラスターの登録

## 概要

クラスターを登録するには、2つの特定のスタイルがあります。これらのスタイルは**エージェント開始**と**マネージャー開始**登録と呼ばれます。通常はエージェント開始登録を選択しますが、特定のユースケースではマネージャー開始の方が適したワークフローとなる場合があります。

### エージェント開始登録

エージェント開始とは、ダウンストリームクラスターが[クラスター登録トークン](#create-cluster-registration-tokens)とオプションでクライアントIDを使用してエージェントをインストールするパターンを指します。クラスターエージェントはその後、FleetマネージャーにAPIリクエストを行い、登録プロセスを開始します。このプロセスを使用すると、マネージャーはダウンストリームクラスターに対してアウトバウンドAPIリクエストを行うことがなく、直接的なネットワークアクセスも不要です。ダウンストリームクラスターはマネージャーに対してアウトバウンドHTTPSコールを行うだけで済みます。

### マネージャー開始登録

マネージャー開始登録は、既存のKubernetesクラスターをFleetマネージャーに登録し、FleetマネージャーがダウンストリームクラスターにAPIコールを行ってエージェントをデプロイするプロセスです。このスタイルでは、Fleetマネージャーが登録プロセスのためにダウンストリームクラスターのAPIサーバーと通信できる必要があるため、追加のネットワークアクセス要件が発生する可能性があります。クラスターが登録された後は、マネージャーがダウンストリームクラスターAPIに連絡する必要はありません。このスタイルは、[cluster-api](https://github.com/kubernetes-sigs/cluster-api)や[Rancher](https://github.com/rancher/rancher)のようなGitOpsを使用してすべてのKubernetesクラスターの作成を管理したい場合により適しています。

## エージェント開始

ダウンストリームクラスターは、Helmを使用してエージェントをインストールし、**クラスター登録トークン**とオプションで**クライアントID**または**クラスターラベル**を使用して登録されます。

:::info
[マルチクラスターの設定](./installation.md#configuration-for-multi-cluster)のためにFleetマネージャーを設定する必要はありません。Helmを介してインストールするダウンストリームエージェントは、直接アップストリームクラスターのKubernetes APIに接続します。

エージェント開始登録は通常、Rancherでは使用されません。
:::

### クラスター登録トークンとクライアントID

**クラスター登録トークン**は、ダウンストリームクラスターエージェントが登録プロセスを開始するための認証情報です。これは必須です。[クラスター登録トークン](./architecture.md#security)は`values.yaml`ファイルとして表現され、`helm install`プロセスに渡されます。あるいは、トークンを直接`--set token="$token"`としてhelm installコマンドに渡すこともできます。

エージェントを登録するには2つのスタイルがあります。このエージェントのためにクラスターを動的に作成する場合、登録時に**クラスターラベル**を指定することをお勧めします。または、Fleetマネージャーで事前定義されたクラスターにエージェントを登録する場合は、**クライアントID**が必要です。前者のアプローチが通常は最も簡単です。

### 新しいクラスターのためのエージェントのインストール

FleetエージェントはHelmチャートとしてインストールされます。以下はそのパラメータを決定し設定する方法の説明です。

まず、[クラスター登録トークンの指示](#create-cluster-registration-tokens)に従って、Fleetクラスターに対して認証するための登録トークンを含む`values.yaml`を取得します。

次に、オプションで、登録時に新しく作成されるクラスターに割り当てられるラベルを定義できます。登録が完了した後、エージェントはクラスターのラベルを変更することはできません。クラスターラベルを追加するには、以下のHelmコマンドに`--set-string labels.KEY=VALUE`を追加します。ラベル`foo=bar`と`bar=baz`を追加するには、コマンドラインに`--set-string labels.foo=bar --set-string labels.bar=baz`を追加します。

```shell
# ラベルが不要な場合は空白のままにします
CLUSTER_LABELS="--set-string labels.example=true --set-string labels.env=dev"
```

次に、ダウンストリームクラスターが接続するために使用するFleetクラスターのAPIサーバーURLとCAの変数を設定します。

```shell
API_SERVER_URL=https://...
API_SERVER_CA_DATA=...
```

`API_SERVER_CA_DATA`の値は、アップストリームクラスターに接続するための有効なデータを含む`.kube/config`ファイルから取得できます（`certificate-authority-data`キーの下）。あるいは、アップストリームクラスター自体から、デフォルトのServiceAccountシークレット名（通常は`default-token-`で始まる、デフォルトのネームスペース内）を調べて、`ca.crt`キーの下から取得することもできます。

:::caution

__適切なネームスペースとリリース名を使用する__:
エージェントチャートのネームスペースは`cattle-fleet-system`で、リリース名は`fleet-agent`である必要があります。

:::

:::warning Kubectlコンテキスト

__正しいクラスターにインストールしていることを確認する__:
Helmは`${HOME}/.kube/config`のデフォルトコンテキストを使用してエージェントをデプロイします。`--kubeconfig`と`--kube-context`を使用して、Helmがインストールするクラスターを変更します。

:::

:::caution RancherのFleet
RancherにはFleet用の別のHelmチャートがあり、異なるリポジトリを使用します。
:::

FleetのHelmリポジトリを追加します。
<CodeBlock language="bash">
{`helm repo add fleet https://rancher.github.io/fleet-helm-charts/`}
</CodeBlock>

最後に、Helmを使用してエージェントをインストールします。
<Tabs>
  <TabItem value="helm" label="インストール" default>
<CodeBlock language="bash">
{`helm -n cattle-fleet-system install --create-namespace --wait \\
    $CLUSTER_LABELS \\
    --values values.yaml \\
    --set apiServerCA="$API_SERVER_CA_DATA" \\
    --set apiServerURL="$API_SERVER_URL" \\
    fleet-agent fleet/fleet-agent`}
</CodeBlock>
</TabItem>
<TabItem value="validate" label="検証">
Fleetポッドのステータスを以下のコマンドで確認できます。

```shell
# kubectlが正しいクラスターを指していることを確認します
kubectl -n cattle-fleet-system logs -l app=fleet-agent
kubectl -n cattle-fleet-system get pods -l app=fleet-agent
```
</TabItem>
</Tabs>
エージェントがデプロイされました。

さらに、新しいクラスターがFleetマネージャーに登録されているはずです。以下は、新しいクラスターが`clusters`[ネームスペース](./namespaces.md)に登録されていることを確認する例です。このコマンドを実行するには、`${HOME}/.kube/config`がFleetマネージャーを指していることを確認してください。

```shell
kubectl -n clusters get clusters.fleet.cattle.io
```
```
NAME                   BUNDLES-READY   NODES-READY   SAMPLE-NODE             LAST-SEEN              STATUS
cluster-ab13e54400f1   1/1             1/1           k3d-cluster2-server-0   2020-08-31T19:23:10Z
```

### 事前定義されたクラスターのためのエージェントのインストール

クライアントIDは、既存のラベルやリポジトリがターゲットとなるFleetマネージャーでクラスターを事前定義するためのものです。クライアントIDは必須ではなく、クラスター管理の一つのアプローチに過ぎません。
**クライアントID**はクラスターを識別するための一意の文字列です。この文字列はユーザーが生成し、Fleetマネージャーやエージェントにとっては不透明です。十分に一意であると想定されます。セキュリティ上の理由から、この値を簡単に推測できないようにする必要があります。そうでないと、あるクラスターが他のクラスターになりすますことができてしまいます。クライアントIDはオプションであり、指定しない場合は`kube-system`ネームスペースリソースのUIDフィールドがクライアントIDとして使用されます。登録時にクライアントIDがFleetマネージャーの`Cluster`リソースに見つかった場合、そのエージェントはその`Cluster`に関連付けられます。クライアントIDを持つ`Cluster`リソースが見つからない場合は、そのクライアントIDを持つ新しい`Cluster`リソースが作成されます。

FleetエージェントはHelmチャートとしてインストールされます。Helmチャートのインストールに必要なパラメータは、クラスター登録トークン（`values.yaml`ファイルで表現される）とクライアントIDだけです。クライアントIDはオプションです。

まず、ランダムに選んだクライアントIDでFleetマネージャーに`Cluster`を作成します。

```yaml
kind: Cluster
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: my-cluster
  namespace: clusters
spec:
  clientID: "really-random"
```

次に、[クラスター登録トークンの指示](#create-cluster-registration-tokens)に従って、使用する`values.yaml`ファイルを取得します。

次に、クライアントIDを使用するように環境を設定します。

```shell
CLUSTER_CLIENT_ID="really-random"
```

:::note

__適切なネームスペースとリリース名を使用する__:
エージェントチャートのネームスペースは`cattle-fleet-system`で、リリース名は`fleet-agent`である必要があります。

:::

:::note

__正しいクラスターにインストールしていることを確認する__:
Helmは`${HOME}/.kube/config`のデフォルトコンテキストを使用してエージェントをデプロイします。`--kubeconfig`と`--kube-context`を使用して、Helmがインストールするクラスターを変更します。

:::

FleetのHelmリポジトリを追加します。
<CodeBlock language="bash">
{`helm repo add fleet https://rancher.github.io/fleet-helm-charts/`}
</CodeBlock>

最後に、Helmを使用してエージェントをインストールします。

<Tabs>
  <TabItem value="helm2" label="インストール" default>
<CodeBlock language="bash">
{`helm -n cattle-fleet-system install --create-namespace --wait \\
    --set clientID="$CLUSTER_CLIENT_ID" \\
    --values values.yaml \\
    fleet-agent fleet/fleet-agent`}
</CodeBlock>

</TabItem>
<TabItem value="validate2" label="検証">
Fleetポッドのステータスを以下のコマンドで確認できます。

```shell
# kubectlが正しいクラスターを指していることを確認します
kubectl -n cattle-fleet-system logs -l app=fleet-agent
kubectl -n cattle-fleet-system get pods -l app=fleet-agent
```
</TabItem>
</Tabs>
エージェントがデプロイされました。

さらに、新しいクラスターがFleetマネージャーに登録されているはずです。以下は、新しいクラスターが`clusters`[ネームスペース](./namespaces.md)に登録されていることを確認する例です。このコマンドを実行するには、`${HOME}/.kube/config`がFleetマネージャーを指していることを確認してください。

```shell
kubectl -n clusters get clusters.fleet.cattle.io
```
```
NAME                   BUNDLES-READY   NODES-READY   SAMPLE-NODE             LAST-SEEN              STATUS
my-cluster             1/1             1/1           k3d-cluster2-server-0   2020-08-31T19:23:10Z
```

### クラスター登録トークンの作成

:::info

__マネージャー開始登録には不要__:
```
マネージャーが開始する登録の場合、トークンはフリートマネージャーによって管理され、手動で作成および取得する必要はありません。

:::

エージェントが開始する登録の場合、ダウンストリームクラスターは[クラスター登録トークン](./architecture.md#security)を持っている必要があります。クラスター登録トークンは、クラスターの新しいアイデンティティを確立するために使用されます。内部的には、クラスター登録トークンは特定のネームスペース内で`ClusterRegistrationRequests`を作成する権限を持つKubernetesサービスアカウントを作成することで管理されます。クラスターが登録されると、そのクラスターのための新しい`ServiceAccount`が作成され、クラスターの一意のアイデンティティとして使用されます。エージェントは登録後にクラスター登録トークンを忘れるように設計されています。エージェントは成功した登録後にクラスター登録トークンの参照を保持しませんが、通常は他のシステムブートストラップスクリプトが保持することに注意してください。

クラスター登録トークンが忘れられるため、クラスターを再登録する必要がある場合は、新しい登録トークンをクラスターに与える必要があります。

#### トークンのTTL

クラスター登録トークンは、ネームスペース内の任意のクラスターによって再利用できます。トークンにはTTLを設定でき、特定の時間後に期限切れになります。

#### 新しいトークンの作成

`ClusterRegistrationToken`はネームスペース型であり、`GitRepo`および`ClusterGroup`リソースを作成するのと同じネームスペースで作成する必要があります。フリートでネームスペースがどのように使用されるかの詳細については、[ネームスペース](./namespaces.md)に関するドキュメントを参照してください。以下のYAMLで新しいトークンを作成します。

```yaml
kind: ClusterRegistrationToken
apiVersion: "fleet.cattle.io/v1alpha1"
metadata:
    name: new-token
    namespace: clusters
spec:
    # このトークンが有効である期間の文字列。値が<= 0またはnullの場合、無限の時間を意味します。
    ttl: 240h
```

`ClusterRegistrationToken`が作成されると、フリートは同じ名前の対応する`Secret`を作成します。`Secret`の作成は非同期で行われるため、使用可能になるまで待つ必要があります。

以下のワンライナーを使用して待つことができます：
```shell
while ! kubectl --namespace=clusters  get secret new-token; do sleep 5; done
```

#### トークン値の取得（Agent values.yaml）

トークン値には、ダウンストリームクラスターにフリートエージェントをインストールするために`helm install`に渡すことが期待される`values.yaml`ファイルのYAMLコンテンツが含まれています。

この値は、上記の`Secret`の`values`フィールドに含まれています。上記の例のYAMLコンテンツを取得するには、以下のワンライナーを実行できます：
```shell
kubectl --namespace clusters get secret new-token -o 'jsonpath={.data.values}' | base64 --decode > values.yaml
```

`values.yaml`が準備できたら、TTLが切れるまでクラスターによって繰り返し使用できます。

## マネージャーが開始する登録

マネージャーが開始する登録フローは、データフィールドに有効なkubeconfigファイルを含むKubernetes`Secret`を参照するフリートマネージャー内の`Cluster`リソースを作成することで実現されます。

:::info
フリートをRancherなしでスタンドアロンで使用する場合は、[インストールの詳細](./installation.md#configuration-for-multi-cluster)に記載されているようにインストールする必要があります。

マネージャーが開始する登録は、Rancherダッシュボードからクラスターを追加する際に使用されます。
:::

### Kubeconfig Secretの作成

このシークレットの形式は、[cluster-api](https://github.com/kubernetes-sigs/cluster-api)で使用されるkubeconfigシークレットの[形式](https://cluster-api.sigs.k8s.io/developer/architecture/controllers/cluster.html#secrets)に一致することを意図しています。これにより、`cluster-api`を使用してフリートに動的に登録されるクラスターを作成できます。

```yaml title="Kubeconfig Secret Example"
kind: Secret
apiVersion: v1
metadata:
  name: my-cluster-kubeconfig
  namespace: clusters
data:
  value: YXBpVmVyc2lvbjogdjEKY2x1c3RlcnM6Ci0gY2x1c3RlcjoKICAgIHNlcnZlcjogaHR0cHM6Ly9leGFtcGxlLmNvbTo2NDQzCiAgbmFtZTogY2x1c3Rlcgpjb250ZXh0czoKLSBjb250ZXh0OgogICAgY2x1c3RlcjogY2x1c3RlcgogICAgdXNlcjogdXNlcgogIG5hbWU6IGRlZmF1bHQKY3VycmVudC1jb250ZXh0OiBkZWZhdWx0CmtpbmQ6IENvbmZpZwpwcmVmZXJlbmNlczoge30KdXNlcnM6Ci0gbmFtZTogdXNlcgogIHVzZXI6CiAgICB0b2tlbjogc29tZXRoaW5nCg==
```

### クラスターリソースの作成

クラスターリソースは、kubeconfigシークレットを参照する必要があります。

```yaml title="Cluster Resource Example"
apiVersion: fleet.cattle.io/v1alpha1
kind: Cluster
metadata:
  name: my-cluster
  namespace: clusters
  labels:
    demo: "true"
    env: dev
spec:
  kubeConfigSecret: my-cluster-kubeconfig
```