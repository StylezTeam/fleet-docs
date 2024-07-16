# ネームスペース

Fleetマネージャーのすべてのタイプはネームスペース化されています。マネージャータイプのネームスペースは、下流クラスターにデプロイされたリソースのネームスペースに対応していません。Fleetマネージャーでネームスペースがどのように使用されるかを理解することは、セキュリティモデルを理解し、Fleetをマルチテナント方式で使用する方法を理解するために重要です。

## GitRepos、Bundles、Clusters、ClusterGroups

主要なタイプはすべてネームスペースにスコープされています。`GitRepo`ターゲットのすべてのセレクターは、同じネームスペース内の`Clusters`および`ClusterGroups`に対して評価されます。これは、ネームスペース内で`GitRepo`タイプに`create`または`update`の権限を与えると、そのエンドユーザーがセレクターを変更してそのネームスペース内の任意のクラスターに一致させることができることを意味します。したがって、2つのチームがそれぞれの`GitRepo`登録を自己管理し、互いのクラスターをターゲットにできないようにする場合、それらは異なるネームスペースに配置する必要があります。

### GitRepoネームスペース

Gitリポジトリは`GitRepo`カスタムリソースタイプを使用してFleetマネージャーに追加されます。`GitRepo`タイプはネームスペース化されています。デフォルトでは、Rancherは2つのFleetワークスペースを作成します：**fleet-default**と**fleet-local**です。

- `Fleet-default`には、すでにRancherを通じて登録されているすべての下流クラスターが含まれます。
- `Fleet-local`には、デフォルトでローカルクラスターが含まれます。

[シングルクラスター](./concepts.md)スタイルでFleetを使用している場合、ネームスペースは常に**fleet-local**になります。`fleet-local`ネームスペースの詳細については[こちら](https://fleet.rancher.io/namespaces/#fleet-local)を参照してください。

[マルチクラスター](./concepts.md)スタイルの場合、正しいターゲットクラスターにマップされる適切なリポジトリを使用するようにしてください。

## バンドルでのネームスペース作成の動作

Fleetバンドルをデプロイする際、指定されたネームスペースが存在しない場合は自動的に作成されます。

## 特殊なネームスペース

Fleetとそのリソースで使用される[ネームスペース](./namespaces.md)の概要。

![Namespace](/img/FleetNamespaces.svg)

### fleet-local（ローカルワークスペース、クラスター登録ネームスペース）

**fleet-local**ネームスペースは、シングルクラスターの使用ケースやFleetマネージャーの設定をブートストラップするために使用される特殊なネームスペースです。

Fleetがインストールされると、`fleet-local`ネームスペースが作成され、`local`という名前の1つの`Cluster`と`default`という名前の1つの`ClusterGroup`が作成されます。`GitRepo`にターゲットが指定されていない場合、デフォルトで`default`という名前の`ClusterGroup`にターゲットされます。これは、`fleet-local`で作成されたすべての`GitRepo`が自動的に`local``クラスター`をターゲットにすることを意味します。`local``クラスター`は、Fleetマネージャーが実行されているクラスターを指します。

クラスター登録ネームスペースには、クラスターとクラスター登録リソース、および任意のgitreposとバンドルが含まれます。

### cattle-fleet-system（システムネームスペース）

FleetコントローラーとFleetエージェントはこのネームスペースで実行されます。`GitRepos`で参照されるすべてのサービスアカウントは、下流クラスターのこのネームスペースに存在することが期待されます。

### cattle-fleet-clusters-system（システム登録ネームスペース）

このネームスペースには、クラスター登録プロセスのためのシークレットが格納されます。特にシークレットを含む他のリソースは含まれないようにする必要があります。

### クラスターネームスペース

登録されたすべてのクラスターに対して、Fleetマネージャーによってそのクラスター用のネームスペースが作成されます。
これらのネームスペースは`cluster-${namespace}-${cluster}-${random}`の形式で命名されます。このネームスペースの目的は、そのクラスターのすべての`BundleDeployments`がこのネームスペースに配置され、下流クラスターがこのネームスペース内の`BundleDeployments`を監視および更新するアクセス権を持つことです。

## クロスネームスペースデプロイメント

ネームスペースをまたいでデプロイするGitRepoを作成することが可能です。これの主な目的は、中央の特権チームが異なるチームによって管理される多くのクラスターの共通設定を管理できるようにすることです。これを実現する方法は、クラスターに`BundleNamespaceMapping`リソースを作成することです。

`BundleNamespaceMapping`リソースを作成する場合、`GitRepos`のみを含むネームスペースで行うのが最善です。`クラスター`が同じリポジトリにあると、クロスネームスペースの`GitRepos`が現在のネームスペースに対して常に評価されるため、混乱することがあります。そのため、同じネームスペースにクラスターがある場合、それらをカナリークラスターにすることを検討するかもしれません。

`BundleNamespaceMapping`には2つのフィールドしかありません。以下の通りです。

```yaml
kind: BundleNamespaceMapping
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: not-important
  namespace: typically-unique

# ラベルで一致するバンドル。ラベルはfleet.yamlの
# labelsフィールドまたはGitRepoのmetadata.labelsフィールドで定義されます
bundleSelector:
  matchLabels:
   foo: bar

# ラベルで一致するネームスペース
namespaceSelector:
  matchLabels:
   foo: bar
```

`BundleNamespaceMappings`の`bundleSelector`フィールドが`Bundles`のラベルに一致する場合、その`Bundle`ターゲット基準は`namespaceSelector`に一致するすべてのネームスペース内のすべてのクラスターに対して評価されます。作成されたバンドルのラベルをgitから指定するには、`fleet.yaml`ファイルまたは`GitRepo`の`metadata.labels`フィールドにラベルを付けます。

## GitReposの制限

ネームスペースには複数の`GitRepoRestriction`リソースを含めることができます。そのネームスペースで作成されたすべての`GitRepos`は、制限リストに対してチェックされます。
`GitRepo`が制約の1つに違反すると、その`BundleDeployment`はエラーステートになり、デプロイされません。

これを使用して、GitRepoの`serviceAccount`および`clientSecretName`フィールドのデフォルトを設定することもできます。

```yaml
kind: GitRepoRestriction
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: restriction
  namespace: typically-unique
allowedClientSecretNames: []
allowedRepoPatterns: []
allowedServiceAccounts: []
allowedTargetNamespaces: []
defaultClientSecretName: ""
defaultServiceAccount: ""
```

### 許可されたターゲットネームスペース

これは、下流クラスターのネームスペースのセットにデプロイを制限するために使用できます。
allowedTargetNamespaces制限が存在する場合、すべての`GitRepos`は`targetNamespace`を指定する必要があり、指定されたネームスペースは許可リストに含まれている必要があります。
これにより、クラスター全体のリソースの作成も防止されます。