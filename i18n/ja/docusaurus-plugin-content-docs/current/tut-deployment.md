import CodeBlock from '@theme/CodeBlock';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# デプロイの作成

ワークロードを下流のクラスターにデプロイするには、まずGitリポジトリを作成し、次にGitRepoリソースを作成して適用します。

このチュートリアルでは、[fleet-examples](https://github.com/rancher/fleet-examples)リポジトリを使用します。

:::note
リポジトリの構造や各バンドルのデプロイを設定する方法の詳細については、[GitRepo Contents](./gitrepo-content.md)を参照してください。
Gitリポジトリごとに利用可能なオプションの詳細については、[Adding a GitRepo](./gitrepo-add.md)を参照してください。
:::

## シングルクラスターの例

すべての例は、クラスターごとのカスタマイズなしでクラスターにコンテンツをデプロイします。これは、Fleet用のGitリポジトリの構造の基本を理解するための良い出発点です。

<Tabs groupId="examples">
<TabItem value="helm" label="Helm" default>

Helmを使用した例です。ローカルクラスターに<a href="https://github.com/rancher/fleet-examples/tree/master/single-cluster/helm">helmの例</a>をデプロイします。

リポジトリには、Helmチャートとデプロイを設定するためのオプションの`fleet.yaml`が含まれています：

```yaml title="fleet.yaml"
namespace: fleet-helm-example

# カスタムHelmオプション
helm:
  # 使用するリリース名。空の場合は生成されたリリース名が使用されます
  releaseName: guestbook

  # リポジトリ内のチャートのディレクトリ。go-getterがサポートする有効なURLも使用できます。
  # 下記のrepoが設定されている場合、この値はリポジトリ内のチャート名です
  chart: ""

  # チャートをダウンロードするための有効なHelmリポジトリのhttps
  repo: ""

  # repoが設定されている場合にチャートのバージョンを検索するために使用されます
  version: ""

  # 更新できないリソースを強制的に再作成
  force: false

  # リリースがアクティブになるまでHelmが待機する時間。値が0以下の場合、Helmでは待機しません
  timeoutSeconds: 0

  # インストールにvalues.yamlとして渡されるカスタム値
  values:
    replicas: 2
```

デプロイを作成するには、カスタムリソースを上流のクラスターに適用します。`fleet-local`ネームスペースにはローカルクラスターリソースが含まれています。ローカルのfleet-agentは`fleet-helm-example`ネームスペースにデプロイを作成します。

```bash
kubectl apply -n fleet-local -f - <<EOF
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - single-cluster/helm
EOF
```

</TabItem>
<TabItem value="helm-multi-chart" label="Helm Multi Chart" default>

単一のリポジトリから複数のチャートをデプロイする<a href="https://github.com/rancher/fleet-examples/blob/master/single-cluster/helm-multi-chart">例</a>です。これは前の例と似ていますが、サブフォルダから3つのHelmチャートをデプロイし、それぞれが独自の`fleet.yaml`で設定されています。

```bash
kubectl apply -n fleet-local -f - <<EOF
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - single-cluster/helm-multi-chart
EOF
```

</TabItem>
<TabItem value="helm-kustomize" label="Helm & Kustomize" default>

<a href="https://github.com/rancher/fleet-examples/blob/master/single-cluster/helm-kustomize">Kustomizeを使用してサードパーティのHelmチャートを修正する</a>例です。
これは、サードパーティソースからダウンロードされたHelmチャートとしてパッケージ化されたKubernetesのサンプルゲストブックアプリケーションをデプロイし、Kustomizeを使用してHelmチャートを修正します。アプリは`fleet-helm-kustomize-example`ネームスペースにデプロイされます。

```bash
kubectl apply -n fleet-local -f - <<EOF
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - single-cluster/helm-kustomize
EOF
```

</TabItem>
<TabItem value="kustomize" label="Kustomize" default>

<a href="https://github.com/rancher/fleet-examples/blob/master/single-cluster/kustomize">Kustomizeを使用した例</a>です。

`fleet.yaml`には、必要な`kustomization.yaml`のパスを指定するための`kustomize:`キーがあることに注意してください：

```yaml title="fleet.yaml"
kustomize:
  # ルートフォルダのものとは異なるkustomization.yamlを使用する場合
  dir: ""
```

```bash
kubectl apply -n fleet-local -f - <<EOF
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - single-cluster/kustomize
EOF
```

</TabItem>
<TabItem value="manifests" label="Manifests" default>

<a href="https://github.com/rancher/fleet-examples/tree/master/single-cluster/manifests">生のKubernetes YAMLを使用した例</a>です。

```bash
kubectl apply -n fleet-local -f - <<EOF
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - single-cluster/manifests
EOF
```

</TabItem>
</Tabs>

## マルチクラスターの例

以下の例では、複数のGitリポジトリを一度に複数のクラスターにデプロイし、各ターゲットに対してアプリを異なる設定でデプロイします。

<Tabs groupId="examples">
<TabItem value="helm" label="Helm" default>

Helmを使用した例です。<a href="https://github.com/rancher/fleet-examples/tree/master/multi-cluster/helm">helmの例</a>をデプロイし、ターゲットクラスターごとにカスタマイズします。

リポジトリには、Helmチャートとデプロイを設定するためのオプションの`fleet.yaml`が含まれています。`fleet.yaml`は、クラスターのラベルに応じて異なるデプロイオプションを設定するために使用されます：

```yaml title="fleet.yaml"
namespace: fleet-mc-helm-example
targetCustomizations:
- name: dev
  helm:
    values:
      replication: false
  clusterSelector:
    matchLabels:
      env: dev

- name: test
  helm:
    values:
      replicas: 3
  clusterSelector:
    matchLabels:
      env: test

- name: prod
  helm:
    values:
      serviceType: LoadBalancer
      replicas: 3
  clusterSelector:
    matchLabels:
      env: prod
```

デプロイを作成するには、カスタムリソースを上流のクラスターに適用します。`fleet-default`ネームスペースには、デフォルトで下流のクラスターリソースが含まれています。チャートは、`targets:`のいずれかのエントリに一致するラベル付きクラスターリソースを持つ`fleet-default`ネームスペース内のすべてのクラスターにデプロイされます。

```yaml title="gitrepo.yaml"
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm
  namespace: fleet-default
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - multi-cluster/helm
  targets:
  - name: dev
    clusterSelector:
      matchLabels:
        env: dev

  - name: test
    clusterSelector:
      matchLabels:
        env: test

  - name: prod
    clusterSelector:
      matchLabels:
        env: prod
```

gitrepoリソースを上流のクラスターに適用することで、Fleetはリポジトリの監視を開始し、デプロイを作成します：

<CodeBlock language="bash">
{`kubectl apply -n fleet-default -f gitrepo.yaml`}
</CodeBlock>

</TabItem>
<TabItem value="helm-external" label="Helm External" default>

<a href="https://github.com/rancher/fleet-examples/blob/master/multi-cluster/helm-external">サードパーティソースからダウンロードされたHelmチャートを使用し、ターゲットクラスターごとにカスタマイズする例</a>です。カスタマイズは前の例と似ています。

デプロイを作成するには、カスタムリソースを上流のクラスターに適用します。`fleet-default`ネームスペースには、デフォルトで下流のクラスターリソースが含まれています。チャートは、`targets:`のいずれかのエントリに一致するラベル付きクラスターリソースを持つ`fleet-default`ネームスペース内のすべてのクラスターにデプロイされます。

```yaml title="gitrepo.yaml"
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm-external
  namespace: fleet-default
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - multi-cluster/helm-external
  targets:
  - name: dev
    clusterSelector:
      matchLabels:
        env: dev

  - name: test
    clusterSelector:
      matchLabels:
        env: test

  - name: prod
    clusterSelector:
      matchLabels:
        env: prod
```

gitrepoリソースを上流のクラスターに適用することで、Fleetはリポジトリの監視を開始し、デプロイを作成します：

<CodeBlock language="bash">
{`kubectl apply -n fleet-default -f gitrepo.yaml`}
</CodeBlock>

</TabItem>
<TabItem value="helm-kustomize" label="Helm & Kustomize" default>

<a href="https://github.com/rancher/fleet-examples/blob/master/multi-cluster/helm-kustomize">Kustomizeを使用してサードパーティのHelmチャートを修正する</a>例です。
これは、サードパーティソースからダウンロードされたHelmチャートとしてパッケージ化されたKubernetesのサンプルゲストブックアプリケーションをデプロイし、Kustomizeを使用してHelmチャートを修正します。アプリは`fleet-helm-kustomize-example`ネームスペースにデプロイされます。

アプリケーションは環境ごとに次のようにカスタマイズされます：

* 開発クラスター：redisリーダーのみがデプロイされ、フォロワーはデプロイされません。
* テストクラスター：フロントデプロイメントを3にスケール
* 本番クラスター：フロントデプロイメントを3にスケールし、サービスタイプをLoadBalancerに設定

`fleet.yaml`は、クラスターのラベルに応じてどのオーバーレイを使用するかを制御するために使用されます：

```yaml title="fleet.yaml"
namespace: fleet-mc-kustomize-example
targetCustomizations:
- name: dev
  clusterSelector:
    matchLabels:
      env: dev
  kustomize:
    dir: overlays/dev

- name: test
  clusterSelector:
    matchLabels:
      env: test
  kustomize:
    dir: overlays/test

- name: prod
  clusterSelector:
    matchLabels:
      env: prod
  kustomize:
    dir: overlays/prod
```

デプロイを作成するには、カスタムリソースを上流のクラスターに適用します。`fleet-default`ネームスペースには、デフォルトで下流のクラスターリソースが含まれています。チャートは、`targets:`のいずれかのエントリに一致するラベル付きクラスターリソースを持つ`fleet-default`ネームスペース内のすべてのクラスターにデプロイされます。

```yaml title="gitrepo.yaml"
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: helm-kustomize
  namespace: fleet-default
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - multi-cluster/helm-kustomize
  targets:
  - name: dev
    clusterSelector:
      matchLabels:
        env: dev

  - name: test
    clusterSelector:
      matchLabels:
        env: test

  - name: prod
    clusterSelector:
      matchLabels:
        env: prod
```

リポジトリリソースをアップストリームクラスターに適用することで、フリートはリポジトリの監視を開始し、デプロイメントを作成します:

<CodeBlock language="bash">
{`kubectl apply -n fleet-default -f gitrepo.yaml`}
</CodeBlock>

</TabItem>
<TabItem value="kustomize" label="Kustomize" default>

<a href="https://github.com/rancher/fleet-examples/blob/master/multi-cluster/kustomize">Kustomizeを使用した例</a>と、ターゲットクラスターごとのカスタマイズ。

`fleet.yaml`のカスタマイズは「Helm & Kustomize」の例と同一です。

デプロイメントを作成するために、カスタムリソースをアップストリームクラスターに適用します。デフォルトでは、`fleet-default`ネームスペースにはダウンストリームクラスターリソースが含まれます。チャートは、`targets:`の下にあるエントリと一致するラベル付きクラスターリソースを持つすべてのクラスターにデプロイされます。

```bash
kubectl apply -n fleet-default -f - <<EOF
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: kustomize
  namespace: fleet-default
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - multi-cluster/kustomize
  targets:
  - name: dev
    clusterSelector:
      matchLabels:
        env: dev

  - name: test
    clusterSelector:
      matchLabels:
        env: test

  - name: prod
    clusterSelector:
      matchLabels:
        env: prod
EOF
```

リポジトリリソースをアップストリームクラスターに適用することで、フリートはリポジトリの監視を開始し、デプロイメントを作成します:

</TabItem>
<TabItem value="manifests" label="Manifests" default>

<a href="https://github.com/rancher/fleet-examples/tree/master/multi-cluster/manifests">生のKubernetes YAMLを使用し、ターゲットクラスターごとにカスタマイズする例</a>。
アプリケーションは環境ごとに次のようにカスタマイズされます:

* 開発クラスター: リーダーのredisのみがデプロイされ、フォロワーはデプロイされません。
* テストクラスター: フロントデプロイメントを3にスケールします。
* 本番クラスター: フロントデプロイメントを3にスケールし、サービスタイプをLoadBalancerに設定します。

`fleet.yaml`は、クラスターのラベルに応じてどの'yaml'オーバーレイが使用されるかを制御します:

```yaml title="fleet.yaml"
namespace: fleet-mc-manifest-example
targetCustomizations:
- name: dev
  clusterSelector:
    matchLabels:
      env: dev
  yaml:
    overlays:
    # overlays/noreplicationフォルダーを参照
    - noreplication

- name: test
  clusterSelector:
    matchLabels:
      env: test
  yaml:
    overlays:
    # overlays/scale3フォルダーを参照
    - scale3

- name: prod
  clusterSelector:
    matchLabels:
      env: prod
  yaml:
    # overlays/servicelb, scale3フォルダーを参照
    overlays:
    - servicelb
    - scale3
```

デプロイメントを作成するために、カスタムリソースをアップストリームクラスターに適用します。デフォルトでは、`fleet-default`ネームスペースにはダウンストリームクラスターリソースが含まれます。チャートは、`targets:`の下にあるエントリと一致するラベル付きクラスターリソースを持つすべてのクラスターにデプロイされます。

デプロイメントを作成するために、カスタムリソースをアップストリームクラスターに適用します。デフォルトでは、`fleet-default`ネームスペースにはダウンストリームクラスターリソースが含まれます。チャートは、`targets:`の下にあるエントリと一致するラベル付きクラスターリソースを持つすべてのクラスターにデプロイされます。

```yaml title="gitrepo.yaml"
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: manifests
  namespace: fleet-default
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - multi-cluster/manifests
  targets:
  - name: dev
    clusterSelector:
      matchLabels:
        env: dev

  - name: test
    clusterSelector:
      matchLabels:
        env: test

  - name: prod
    clusterSelector:
      matchLabels:
        env: prod
```

<CodeBlock language="bash">
{`kubectl apply -n fleet-default -f gitrepo.yaml`}
</CodeBlock>

</TabItem>
</Tabs>
