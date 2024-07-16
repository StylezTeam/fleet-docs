# fleet.yaml

`fleet.yaml` ファイルはバンドルにオプションを追加します。`fleet.yaml` が含まれるディレクトリは自動的にバンドルに変換されます。

`fleet.yaml` を使用してバンドルをカスタマイズする方法の詳細については、[Git Repository Contents](./gitrepo-content.md) を参照してください。

`fleet.yaml` の内容は [pkg/bundlereader/read.go](https://github.com/rancher/fleet/blob/b501b7e7864d37e310dfcdb109c73e5aec4240bb/pkg/bundlereader/read.go#L132-L139) の構造体に対応しており、[BundleSpec](./ref-crds#bundlespec) を含んでいます。

### リファレンス

```yaml title="fleet.yaml"
# リソースに適用されるデフォルトのネームスペース。このフィールドは特定のネームスペースへのデプロイを強制またはロックダウンするために使用されるのではなく、マニフェストにネームスペースフィールドが指定されていない場合にデフォルト値を提供します。
#
# デフォルト: default
defaultNamespace: default

# すべてのリソースはこのネームスペースに割り当てられ、クラスター範囲のリソースが存在する場合、デプロイは失敗します。
#
# デフォルト: ""
namespace: default

# namespaceLabels は Fleet によって作成されたネームスペースに追加されるラベルです。
namespaceLabels:
  key: value

# namespaceAnnotations は Fleet によって作成されたネームスペースに追加されるアノテーションです。
namespaceAnnotations:
  key: value

# バンドルに設定され、dependsOn.selector で使用できるラベルのオプションマップ。
labels:
  key: value

kustomize:
  # kustomize リソース用のカスタムフォルダーを使用します。このフォルダーには kustomization.yaml ファイルが含まれている必要があります。
  dir: ./kustomize

helm:

  # これらのオプションは "fleet apply" がチャートをダウンロードする方法を制御します。
  #
  # Helm チャートのカスタムロケーションを使用します。これは任意の go-getter URL または OCI レジストリベースの Helm チャート URL を参照できます。例: "oci://ghcr.io/fleetrepoci/guestbook"。これにより、ほとんどの場所からチャートをダウンロードできます。また、go-getter URL はダウンロードを検証するためのダイジェストの追加をサポートしています。以下の repo が設定されている場合、このフィールドはチャートを検索するための名前です。
  #
  # Git リポジトリからチャートをダウンロードすることも可能です。例: `git@github.com:rancher/fleet-examples//single-cluster/helm`。GitRepo で `helmSecretName` を介して SSH キーのシークレットが定義されている場合、それはチャート URL に注入されます。
  #
  # Git リポジトリは、例えば `git::http://github.com/rancher/fleet-examples/single-cluster/helm` を使用して認証なしの http 経由でダウンロードできます。
  chart: ./chart

  # チャートをダウンロードするための Helm リポジトリの https URL。通常は `chart` フィールドを使用して tgz ファイルを参照する方が簡単です。repo が使用される場合、`chart` の値は Helm リポジトリで検索するチャート名として使用されます。
  repo: https://charts.rancher.io

  # チャートのバージョンまたはチャートを見つけるための semver 制約。制約が指定されている場合、git が変更されるたびに評価されます。
  #
  # バージョンは OCI レジストリからダウンロードするチャートも決定します。注意: OCI レジストリは semver でサポートされている '+' 文字をサポートしていません。Helm チャートを '+' 文字を含むタグでプッシュする場合、Helm は自動的に '+' を '_' に置き換えてアップロードします。
  #
  # このファイルでは '+' を使用するべきです。'_' 文字は semver でサポートされておらず、Fleet も OCI レジストリにアクセスする際に '+' を '_' に置き換えます。
  version: 0.1.0

  # デフォルトでは、Fleet は Helm チャートに見つかった依存関係をダウンロードします。この機能を無効にするには disableDependencyUpdate: true を使用します。
  disableDependencyUpdate: false

  ### これらのオプションは helm タイプのバンドルにのみ適用されます。
  #
  # インストール中に helm に渡される `values.yaml` に配置されるべき任意の値。
  values:

    any-custom: value

    # Rancher クラスターのすべてのラベルは global.fleet.clusterLabels.LABELNAME を使用して利用可能です。これらは変数として直接アクセスできます。参照されたクラスターラベルがターゲットクラスターに存在しない場合、変数の値は空の文字列になります。
    variableName: global.fleet.clusterLabels.LABELNAME

    # テンプレートに関する詳細は以下のテンプレートノートを参照してください。
    templatedLabel: "${ .ClusterLabels.LABELNAME }-foo"

    valueFromEnv:
      "${ .ClusterLabels.ENV }": ${ .ClusterValues.someValue | upper | quote }

  # インストール中に helm に渡される必要がある任意の values ファイルへのパス。
  valuesFiles:
    - values1.yaml
    - values2.yaml

  # ダウンストリームクラスターで定義された configmaps またはシークレットから values ファイルを使用できるようにします。
  valuesFrom:
    - configMapKeyRef:
        name: configmap-values
        # バンドルのネームスペースにデフォルト
        namespace: default
        key: values.yaml
    - secretKeyRef:
        name: secret-values
        namespace: default
        key: values.yaml

  ### これらのオプションは fleet-agent がバンドルをデプロイする方法を制御します。これらは kustomize および manifest スタイルのバンドルにも適用されます。
  #
  # チャートをデプロイするためのカスタムリリース名。指定されていない場合、GitRepo.name + GitRepo.path を組み合わせてリリース名が生成されます。
  releaseName: my-release
  #
  # Helm の独自のアノテーションのチェックをスキップさせます。
  takeOwnership: false
  #
  # 変更不可能なリソースを上書きします。これは危険です。
  force: false
  #
  # アップグレード時に Helm の --atomic フラグを設定します。
  atomic: false
  #
  # Fleet の値に対する go テンプレートの事前処理を無効にします。
  disablePreProcess: false
  #
  # Helm のテンプレート関数で DNS 解決を無効にします。
  disableDNS: false
  #
  # values.schema.json ファイルの評価をスキップします。
  skipSchemaValidation: false
  #
  # 設定されており、timeoutSeconds が提供されている場合、すべてのジョブが完了するまで待機し、その後 GitRepo を準備完了とマークします。timeoutSeconds の間待機します。
  waitForJobs: true

# 一時停止されたバンドルはダウンストリームクラスターを更新せず、代わりにバンドルを OutOfSync としてマークします。その後、手動でバンドルをダウンストリームクラスターにデプロイすることを確認できます。
#
# デフォルト: false
paused: false

rolloutStrategy:

  # バンドルの更新中に利用できないクラスターの数または割合。これはデプロイのロールアウト戦略と同じ基本アプローチに従います。クラスターの数が利用不可状態に達すると更新が一時停止されます。デフォルト値は 100% で、更新には影響しません。
  #
  # デフォルト: 100%
  maxUnavailable: 15%

  # バンドルの更新中に利用できないクラスターのパーティションの数または割合。
  #
  # デフォルト: 0
  maxUnavailablePartitions: 20%

  # 特定のパーティション戦略が構成されていない場合にクラスターを自動的にパーティション分割する方法の数または割合。
  #
  # デフォルト: 25%
  autoPartitionSize: 10%

  # パーティションの定義のリスト。ターゲットクラスターが構成に一致しない場合、それらは autoPartitionSize に従って最後にパーティションに追加されます。
  partitions:

    # 表示用にパーティションに与えられるユーザーフレンドリーな名前（オプション）。
    # デフォルト: ""
    - name: canary

      # このパーティションで利用不可と見なされるクラスターの数または割合。
      # デフォルト: 10%
      maxUnavailable: 10%

      # このパーティションに含めるクラスターラベルに一致するセレクター。
      clusterSelector:
        matchLabels:
          env: prod

      # このパーティションに含めるクラスターグループ名。
      clusterGroup: agroup

      # このパーティションに含めるクラスターグループラベルに一致するセレクター。
      clusterGroupSelector:
        clusterSelector:
          matchLabels:
            env: prod

# ターゲットカスタマイズは、ターゲットごとにリソースをどのように変更するかを決定するために使用されます。ターゲットは順番に評価され、クラスターに一致する最初のターゲットがそのクラスターに使用されます。
targetCustomizations:

  # ターゲットの名前。指定されていない場合、デフォルトの名前 "target000" が使用されます。この値は主に表示用です。
  - name: prod

    # ルートの値を上書きするカスタムネームスペース値。
    namespace: newvalue

    # ルートの値を上書きするカスタム defaultNamespace 値。
    defaultNamespace: newdefaultvalue

    # ルートのオプションを上書きするカスタム kustomize オプション。
    kustomize: {}

    # ルートのオプションを上書きするカスタム Helm オプション。
    helm: {}

    # 生の YAML を使用する場合、リソースを置き換えまたはパッチするために使用される overlays/{name} にマップされる名前。./subdir/resource.yaml ファイルをカスタマイズする場合、./overlays/myoverlay/subdir/resource.yaml ファイルがベースファイルを置き換えます。./overlays/myoverlay/subdir/resource_patch.yaml ファイルはベースファイルをパッチします。パッチは JSON Patch または JSON Merge 形式、または組み込みの Kubernetes タイプの戦略的マージパッチである可能性があります。詳細については「Raw YAML Resource Customization」を参照してください。
    yaml:
      overlays:
        - custom2
        - custom3

    # クラスターを一致させるために使用されるセレクター。構造は標準の metav1.LabelSelector 形式です。clusterGroupSelector または clusterGroup が指定されている場合、clusterSelector は clusterGroupSelector および clusterGroup が評価された後に選択をさらに絞り込むためにのみ使用されます。
    clusterSelector:
      matchLabels:
        env: prod

    # 名前で特定のクラスターに一致させるために使用されるセレクター。Rancher で Fleet を使用する場合、clusters.fleet.cattle.io リソースの名前を必ず入力してください。
    clusterName: dev-cluster

    # クラスターグループに一致させるために使用されるセレクター。
    clusterGroupSelector:
      matchLabels:
        region: us-east

    # 選択される特定の clusterGroup の名前。
    clusterGroup: group1

    # doNotDeploy が true の場合、一致するクラスターにリソースはデプロイされません。
    doNotDeploy: false

    # ドリフト修正は、Fleet によって管理されるリソースに対して行われた外部の変更を削除します。デフォルトで 3 方向マージ戦略を使用する helm ロールバックを実行します。force が有効になっている場合、すべてのリソースを更新しようとします。3 方向の戦略的マージは、配列内のアイテムを更新しようとすると失敗する可能性があります。
    # 既存のものを置き換えます。これはforceを使用することで修正できます。forceが有効になっている場合、リソースが再作成される可能性があることに注意してください。keepFailHistoryがtrueに設定されていない限り、失敗したロールバックはhelmの履歴から削除されます。
    correctDrift:
      enabled: false
      force: false # 警告: trueに設定するとリソースが再作成される可能性があります
      keepFailHistory: false

# dependsOnを使用すると、他のバンドルへの依存関係を設定できます。現在のバンドルは、すべての依存関係がデプロイされ、Ready状態になった後にのみデプロイされます。
dependsOn:

  # フォーマット:
  #     <GITREPO-NAME>-<BUNDLE_PATH> すべてのパスセパレータを "-" に置き換えます
  #
  # 例:
  #
  #      GitRepo名 "one", バンドルパス "/multi-cluster/hello-world"
  #      結果は "one-multi-cluster-hello-world" になります。
  #
  # 注意:
  #
  #   バンドル名は53文字以内に制限されています。長い場合は短縮されます:
  #
  #     opni-fleet-examples-fleets-opni-ui-plugin-operator-crd は
  #     opni-fleet-examples-fleets-opni-ui-plugin-opera-021f7 になります
  - name: one-multi-cluster-hello-world

  # ラベルに基づいて依存するバンドルを選択します。
  - selector:
      matchLabels:
        app: weak-monkey

# バンドルを監視する際にフィールドを無視します。これは、Fleetがカスタムリソースのいくつかの条件によりバンドルがエラーステートにあると誤認する場合に使用できます。
ignore:

  # 無視する条件
  conditions:

    # この例では、条件が {"type": "Active", "status", "False"} を含む場合に無視されます
    - type: Active
      status: "False"

# GitRepoで定義されたターゲットを上書きします。overrideTargetsが提供されている場合、バンドルはGitRepoからのターゲットを持ちません。
overrideTargets:
  - clusterSelector:
      matchLabels:
        env: dev
```

### Helmオプション

#### fleet-agentがバンドルをデプロイする方法

これらのオプションは、kustomizeおよびマニフェストスタイルのバンドルにも適用されます。これらは、fleet-agentがバンドルをデプロイする方法を制御します。すべてのバンドルはHelmチャートに変換され、Helm SDKを使用してデプロイされます。これらのオプションは、インストールおよび更新のためのHelm CLIオプションに似ています。

- releaseName
- takeOwnership
- force
- atomic
- disablePreProcess
- disableDNS
- skipSchemaValidation
- waitForJobs

#### Helmチャートダウンロードオプション

これらのオプションはHelmスタイルのバンドル用で、チャートのダウンロード方法を指定します。

- chart
- repo
- version

チャートへの参照は次のいずれかです:

- クローンされたGitリポジトリ内のローカルパス、`chart`で指定。
- [go-getter URL](https://github.com/hashicorp/go-getter?tab=readme-ov-file#url-format)で指定された`chart`。これを使用してチャートのtarballをダウンロードできます。go-getterはGitリポジトリからチャートをダウンロードすることもできます。
- Helmリポジトリ、`repo`およびオプションで`version`で指定。
- OCI Helmリポジトリ、`repo`およびオプションで`version`で指定。

#### Helmチャート値オプション

ダウンロードされたHelmチャートのオプション。

- values
- valuesFiles
- valueFrom

### テンプレート

高度なテンプレートニーズのために、キーと値をgoテンプレート文字列として指定することが可能です。ほとんどの関数は[sprigテンプレートライブラリ](https://masterminds.github.io/sprig/)から利用可能です。

関数の出力が毎回変わる場合、例えば`uuidv4`、バンドルは再デプロイされます。

テンプレートコンテキストには次のキーがあります:

* `.ClusterValues`はターゲットクラスターの`spec.templateValues`から取得されます
* `.ClusterLabels`および`.ClusterAnnotations`はクラスターリソースのラベルおよびアノテーションです。
* `.ClusterName`はfleetのクラスターリソース名です。
* `.ClusterNamespace`はクラスターリソースが存在するネームスペースです。

ラベルまたはアノテーションにキー名でアクセスするには:

```
${ get .ClusterLabels "management.cattle.io/cluster-display-name" }
```

注意: fleet.yamlは有効なyamlでなければなりません。テンプレートは`${ }`をデリミタとして使用しますが、Helmは`{{ }}`を使用します。これらのfleet.yamlテンプレートデリミタはバックティックでエスケープできます。例:

```
foo-bar-${`${PWD}`}
```

は次のテキストになります:

```
foo-bar-${PWD}
```