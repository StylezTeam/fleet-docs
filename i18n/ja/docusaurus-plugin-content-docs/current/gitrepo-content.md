# Git リポジトリの内容

Fleet は git リポジトリからバンドルを作成します。これはパスを指定するか、`fleet.yaml` が見つかった場合に明示的に行われます。

各バンドルは GitRepo のパスから作成され、発見された `fleet.yaml` ファイルを読み取ることでさらに修正されます。
バンドルのライフサイクルは、各バンドルに追加された helm の releaseName フィールドによってリリース間で追跡されます。releaseName が
fleet.yaml 内で指定されていない場合、`GitRepo.name + path` から生成されます。長い名前は切り捨てられ、`-<hash>` プレフィックスが追加されます。

**Git リポジトリには明示的に必要な構造はありません。** スキャンされたリソースは Kubernetes のリソースとして保存されるため、
git でスキャンするディレクトリに任意の大きさのリソースが含まれていないことを確認することが重要です。現在、デプロイされるリソースは **gzip で 1MB 未満** である必要があります。

## リポジトリのスキャン方法

複数のパスを `GitRepo` に定義でき、各パスは独立してスキャンされます。
内部的には、スキャンされた各パスは Fleet が管理、デプロイ、監視する [バンドル](./concepts.md) になります。

リソースがどのようにデプロイされるかを決定するために、次のファイルが探されます。

| ファイル | 場所 | 意味 |
|------|----------|---------|
| **Chart.yaml**:| `path` に対して相対的、または `fleet.yaml` からのカスタムパス | リソースは Helm チャートとしてデプロイされます。詳細は `fleet.yaml` を参照してください。 |
| **kustomization.yaml**:| `path` に対して相対的、または `fleet.yaml` からのカスタムパス | リソースは Kustomize を使用してデプロイされます。詳細は `fleet.yaml` を参照してください。 |
| **fleet.yaml** | 任意のサブパス | 任意の fleet.yaml が見つかった場合、新しい [バンドル](./concepts.md) が定義されます。これにより、同じリポジトリ内でチャート、kustomize、および生の YAML を混在させることができます。 |
| ** *.yaml ** | 任意のサブパス | `Chart.yaml` または `kustomization.yaml` が見つからない場合、任意の `.yaml` または `.yml` ファイルが Kubernetes リソースと見なされ、デプロイされます。 |
| **overlays/{name}** | `path` に対して相対的 | 生の YAML を使用してデプロイする場合（Kustomize や Helm を使用しない場合）、`overlays` はカスタマイズ用の特別なディレクトリです。 |

### バンドルからファイルとディレクトリを除外する

Fleet は `.fleetignore` ファイルを使用してファイルとディレクトリの除外をサポートしています。これは git リポジトリの `.gitignore` ファイルと同様に動作します:
* Golang の [`filepath.Match`](https://pkg.go.dev/path/filepath#Match) を使用して、ファイルまたはディレクトリを一致させるためにグロブ構文が使用されます。
* 空行はスキップされるため、読みやすさを向上させるために使用できます。
* 空白や `#` などの文字はバックスラッシュでエスケープできます。
* 末尾のスペースはエスケープされない限り無視されます。
* コメント、つまりエスケープされていない `#` で始まる行はスキップされます。
* 指定された行は、セパレータが提供されていなくてもファイルまたはディレクトリに一致する可能性があります。例: `subdir/*` と `subdir` は両方とも有効な `.fleetignore` 行であり、`subdir` は `subdir` という名前のファイルとディレクトリの両方に一致します。
* `.fleetignore` が存在するディレクトリの下の任意のレベルでファイルまたはディレクトリに一致する可能性があります。例: `foo.yaml` は `./foo.yaml` および `./path/to/foo.yaml` に一致します。
* 複数の `.fleetignore` ファイルがサポートされています。たとえば、次のディレクトリ構造では、`root/something.yaml`、`bar/something2.yaml`、および `foo/something.yaml` のみがバンドルに含まれます:
```
root/
├── .fleetignore            # `ignore-always.yaml` を含む
├── something.yaml
├── bar
│   ├── .fleetignore        # `something.yaml` を含む
│   ├── ignore-always.yaml
│   ├── something2.yaml
│   └── something.yaml
└── foo
    ├── ignore-always.yaml
    └── something.yaml
```

現在、次の制限があります。以下はサポートされていません:
* ダブルアスタリスク (`**`)
* `!` を使用した明示的な包含

## `fleet.yaml`

`fleet.yaml` は、リソースのデプロイ方法とカスタマイズ方法を変更するために git リポジトリに含めることができるオプションのファイルです。`fleet.yaml` は常に `GitRepo` の `path` に対してルートにあり、サブディレクトリに `fleet.yaml` が見つかった場合、新しい [バンドル](./concepts.md) が定義され、親バンドルとは異なる設定が行われます。

:::caution

__Helm チャートの依存関係__:
Helm チャートの依存関係リストを満たすのはユーザーの責任です。そのため、インストール前に手動で `helm dependencies update $chart` または `helm dependencies build $chart` を実行する必要があります。詳細は Rancher の [Fleet ドキュメント](https://rancher.com/docs/rancher/v2.6/en/deploy-across-clusters/fleet/#helm-chart-dependencies) を参照してください。

:::

利用可能なフィールドは [fleet.yaml リファレンス](./ref-fleet-yaml.md) に記載されています。

プライベート Helm リポジトリの場合、ユーザーは git リポジトリリソースからシークレットを参照できます。
詳細は [プライベート Helm リポジトリの使用](./gitrepo-add.md#using-private-helm-repositories) を参照してください。

## Helm Values の使用

__`values.yaml` に対する変更の適用方法__:

- `values.yaml` に対して最も最近適用された変更が、以前に存在していた値を上書きします。

- 複数のソースから同時に `values.yaml` に変更が適用される場合、値は次の順序で更新されます: `helm.values` -> `helm.valuesFiles` -> `helm.valuesFrom`。つまり、`valuesFrom` が `valuesFiles` および `values` の両方よりも優先されます。

### ValuesFrom の使用

これらの例は、`valuesFrom` の使用スタイルと形式を示しています。ConfigMaps と Secrets は *ダウンストリームクラスター* で作成する必要があります。

[ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) の例:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: configmap-values
  namespace: default
data:
  values.yaml: |-
    replication: true
    replicas: 2
    serviceType: NodePort
```

[Secret](https://kubernetes.io/docs/concepts/configuration/secret/) の例:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: secret-values
  namespace: default
stringData:
  values.yaml: |-
    replication: true
    replicas: 3
    serviceType: NodePort
```

このようなシークレットは、YAML ファイル `secretdata.yaml` から次の kubectl コマンドを実行して作成できます: `kubectl create secret generic secret-values --from-file=values.yaml=secretdata.yaml`

その後、リソースは `fleet.yaml` から参照できます:

```yaml
helm:
  chart: simple-chart
  valuesFrom:
    - secretKeyRef:
        name: secret-values
        namespace: default
        key: values.yaml
    - configMapKeyRef:
        name: configmap-values
        namespace: default
        key: values.yaml
  values:
    replicas: "4"
```

## クラスターごとのカスタマイズ

`GitRepo` は git リポジトリをデプロイするクラスターを定義し、リポジトリ内の `fleet.yaml` はターゲットごとにリソースをカスタマイズする方法を決定します。

`GitRepo` と同じ名前空間にあるすべてのクラスターとクラスターグループは、その `GitRepo` のすべてのターゲットに対して評価されます。ターゲットリストは一つずつ評価され、一致があればリソースはクラスターにデプロイされます。
`GitRepo` のターゲットリストに一致しない場合、リソースはそのクラスターにデプロイされません。
ターゲットクラスターが一致すると、git リポジトリの `fleet.yaml` がカスタマイズのために参照されます。
`fleet.yaml` の `targetCustomizations` は一つずつ評価され、最初の一致がリソースの設定方法を定義します。一致がない場合、リソースは追加のカスタマイズなしでデプロイされます。

`GitRepo` の `targets` および `fleet.yaml` の `targetCustomizations` の両方でクラスターを一致させるためのアプローチは三つあります。
クラスターセレクター、クラスターグループセレクター、または明示的なクラスターグループ名を使用できます。すべての基準は加算されるため、最終的な一致は「clusterSelector && clusterGroupSelector && clusterGroup」として評価されます。三つのうちのいずれかがデフォルト値を持つ場合、それは基準から除外されます。デフォルト値は null または "" です。セレクターの値 `{}` は「すべてに一致する」ことを意味します。

```yaml
targetCustomizations:
- name: all
  # すべてに一致
  clusterSelector: {}
- name: none
  # セレクター無視
  clusterSelector: null
```

クラスター名で一致させる場合は、`clusters.fleet.cattle.io` リソースの名前を使用してください。Rancher UI にはプロビジョニングクラスターリソースと管理クラスターリソースもあります。管理クラスターリソースは名前空間がないため、その名前は異なり、ランダムなサフィックスが含まれます。

```yaml
targetCustomizations:
- name: prod
  clusterName: fleetname
```

詳細およびサポートされているカスタマイズのリストについては、[ダウンストリームクラスターへのマッピング](gitrepo-targets#customization-per-cluster) を参照してください。

## 生の YAML リソースのカスタマイズ

Kustomize または Helm を使用する場合、`kustomization.yaml` または `helm.values` がターゲットクラスターごとにリソースをカスタマイズする方法を制御します。生の YAML を使用する場合、次のシンプルなメカニズムが組み込まれており、使用できます。git リポジトリの `overlays/` フォルダーは、ターゲットクラスターごとにオーバーレイを選択するための特別なフォルダーとして扱われます。リソースオーバーレイの内容はファイル名ベースのアプローチを使用します。これは、リソースベースのアプローチを使用する kustomize とは異なります。kustomize では、リソースのグループ、種類、バージョン、名前、および名前空間がリソースを識別し、それらがマージまたはパッチされます。Fleet では、オーバーレイリソースは一致するファイル名のコンテンツを上書きまたはパッチします。

```shell
# ベースファイル
deployment.yaml
svc.yaml

# オーバーレイファイル

# 次のファイルが追加されます
overlays/custom/configmap.yaml
# 次のファイルが svc.yaml を置き換えます
overlays/custom/svc.yaml
# 次のファイルが deployment.yaml をパッチします
overlays/custom/deployment_patch.yaml
```

`foo` という名前のファイルは、ベースリソースまたは以前のオーバーレイから `foo` という名前のファイルを置き換えます。ファイルの内容をパッチするために、ファイル名に `_patch.`（末尾のピリオドに注意）を追加するという慣例が使用されます。ファイル名から `_patch.` という文字列が `.` に置き換えられ、それがターゲットとして使用されます。たとえば、`deployment_patch.yaml` は `deployment.yaml` をターゲットにします。パッチは JSON マージ、ストラテジックマージパッチ、または JSON パッチを使用して適用されます。
どの戦略が使用されるかはファイルの内容に基づいています。JSON戦略が使用される場合でも、ファイルはYAML構文を使用して書くことができます。

## クラスターとバンドルの状態

[クラスターとバンドルの状態](./cluster-bundles-state.md)を参照してください。

## ネストされたGitRepo CRs

ネストされた`GitRepo CRs`（1つ以上の`GitRepo`リソースを含むリポジトリを指す`GitRepo`を定義すること）はサポートされています。
この機能を使用して、`GitRepo`リソースで`GitOps`を活用したり、複雑なシナリオを複数の`GitRepo`リソースに分割したりすることができます。
`Bundle`内で`GitRepo`を見つけた場合、Fleetは他のリソースと同様にそれをデプロイします。

[この例](https://github.com/rancher/fleet-examples/tree/master/single-cluster/multi-gitrepo)を参照してください。