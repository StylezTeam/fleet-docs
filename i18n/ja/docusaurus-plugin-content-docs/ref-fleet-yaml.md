# fleet.yaml

`fleet.yaml`ファイルはバンドルにオプションを追加します。`fleet.yaml`が含まれるディレクトリは自動的にバンドルに変換されます。

`fleet.yaml`を使用してバンドルをカスタマイズする方法の詳細については、[Git Repository Contents](./gitrepo-content.md)を参照してください。

`fleet.yaml`の内容は、[pkg/bundlereader/read.go](https://github.com/rancher/fleet/blob/b501b7e7864d37e310dfcdb109c73e5aec4240bb/pkg/bundlereader/read.go#L132-L139)の構造体に対応しており、[BundleSpec](./ref-crds#bundlespec)を含んでいます。

### リファレンス

```yaml title="fleet.yaml"
# リソースに適用されるデフォルトのネームスペース。このフィールドは特定のネームスペースへのデプロイを強制またはロックダウンするために使用されるのではなく、マニフェストでネームスペースフィールドが指定されていない場合にデフォルト値を提供します。
#
# デフォルト: default
defaultNamespace: default

# すべてのリソースはこのネームスペースに割り当てられ、クラスター範囲のリソースが存在する場合、デプロイは失敗します。
#
# デフォルト: ""
namespace: default

# namespaceLabelsは、Fleetによって作成されたネームスペースに追加されるラベルです。
namespaceLabels:
  key: value

# namespaceAnnotationsは、Fleetによって作成されたネームスペースに追加されるアノテーションです。
namespaceAnnotations:
  key: value

# バンドルに設定され、dependsOn.selectorで使用できるラベルのオプションマップ。
labels:
  key: value

kustomize:
  # kustomizeリソース用のカスタムフォルダーを使用します。このフォルダーにはkustomization.yamlファイルが含まれている必要があります。
  dir: ./kustomize

helm:

  # これらのオプションは"fleet apply"がチャートをダウンロードする方法を制御します。
  #
  # Helmチャートのカスタムロケーションを使用します。これは任意のgo-getter URLまたはOCIレジストリベースのHelmチャートURLを参照できます。例: "oci://ghcr.io/fleetrepoci/guestbook"。これにより、ほとんどの場所からチャートをダウンロードできます。また、go-getter URLはダウンロードを検証するためのダイジェストを追加することをサポートしています。以下のrepoが設定されている場合、このフィールドはチャートを検索するための名前です。
  #
  # Gitリポジトリからチャートをダウンロードすることも可能です。例: `git@github.com:rancher/fleet-examples//single-cluster/helm`。GitRepoで`helmSecretName`を介してSSHキーのシークレットが定義されている場合、それはチャートURLに注入されます。
  #
  # Gitリポジトリは認証されていないhttpを使用してダウンロードできます。例: `git::http://github.com/rancher/fleet-examples/single-cluster/helm`。
  chart: ./chart

  # チャートをダウンロードするためのHelmリポジトリのhttps URL。通常は`chart`フィールドを使用してtgzファイルを参照する方が簡単です。repoが使用されている場合、`chart`の値はHelmリポジトリで検索するチャート名として使用されます。
  repo: https://charts.rancher.io

  # チャートのバージョンまたはチャートを見つけるためのsemver制約。制約が指定されている場合、gitが変更されるたびに評価されます。
  #
  # バージョンはOCIレジストリからダウンロードするチャートも決定します。注意: OCIレジストリはsemverでサポートされている'+'文字をサポートしていません。Helmチャートを'+'文字を含むタグでプッシュする場合、Helmは自動的に'+'を'_'に置き換えてアップロードします。
  #
  # このファイルでは'+'を使用するべきです。'_'文字はsemverでサポートされておらず、FleetもOCIレジストリにアクセスする際に'+'を'_'に置き換えます。
  version: 0.1.0

  # デフォルトでは、FleetはHelmチャートに見つかった依存関係をダウンロードします。この機能を無効にするにはdisableDependencyUpdate: trueを使用します。
  disableDependencyUpdate: false

  ### これらのオプションはhelmタイプのバンドルにのみ機能します。
  #
  # インストール中にhelmに渡される`values.yaml`に配置されるべき任意の値。
  values:

    any-custom: value

    # Rancherクラスターのすべてのラベルはglobal.fleet.clusterLabels.LABELNAMEを使用して利用可能です。これらは変数として直接アクセスできます。参照されたクラスターラベルがターゲットクラスターに存在しない場合、変数の値は空の文字列になります。
    variableName: global.fleet.clusterLabels.LABELNAME

    # テンプレートの詳細については、以下のテンプレートノートを参照してください。
    templatedLabel: "${ .ClusterLabels.LABELNAME }-foo"

    valueFromEnv:
      "${ .ClusterLabels.ENV }": ${ .ClusterValues.someValue | upper | quote }

  # インストール中にhelmに渡す必要がある任意のvaluesファイルへのパス。
  valuesFiles:
    - values1.yaml
    - values2.yaml

  # ダウンストリームクラスターで定義されたconfigmapsまたはsecretsからvaluesファイルを使用できるようにします。
  valuesFrom:
    - configMapKeyRef:
        name: configmap-values
        # バンドルのネームスペースにデフォルト設定
        namespace: default
        key: values.yaml
    - secretKeyRef:
        name: secret-values
        namespace: default
        key: values.yaml

  ### これらのオプションは、fleet-agentがバンドルをデプロイする方法を制御します。kustomizeおよびマニフェストスタイルのバンドルにも適用されます。
  #
  # チャートをデプロイするためのカスタムリリース名。指定されていない場合、GitRepo.name + GitRepo.pathを組み合わせてリリース名が生成されます。
  releaseName: my-release
  #
  # Helmが独自のアノテーションのチェックをスキップするようにします。
  takeOwnership: false
  #
  # 変更不可能なリソースを上書きします。これは危険です。
  force: false
  #
  # アップグレード時にHelmの--atomicフラグを設定します。
  atomic: false
  #
  # Fleetの値に対するgoテンプレートの事前処理を無効にします。
  disablePreProcess: false
  #
  # Helmのテンプレート関数でDNS解決を無効にします。
  disableDNS: false
  #
  # values.schema.jsonファイルの評価をスキップします。
  skipSchemaValidation: false
  #
  # 設定されており、timeoutSecondsが提供されている場合、すべてのジョブが完了するまで待機し、GitRepoを準備完了としてマークします。timeoutSecondsの間待機します。
  waitForJobs: true

# 一時停止されたバンドルはダウンストリームクラスターを更新せず、代わりにバンドルをOutOfSyncとしてマークします。その後、手動でバンドルをダウンストリームクラスターにデプロイすることを確認できます。
#
# デフォルト: false
paused: false

rolloutStrategy:

  # バンドルの更新中に利用できないクラスターの数または割合。これはデプロイメントのロールアウト戦略と同じ基本アプローチに従います。クラスターが利用できない状態になると、更新は一時停止されます。デフォルト値は100%で、更新には影響しません。
  #
  # デフォルト: 100%
  maxUnavailable: 15%

  # バンドルの更新中に利用できないクラスターのパーティションの数または割合。
  #
  # デフォルト: 0
  maxUnavailablePartitions: 20%

  # 特定のパーティショニング戦略が構成されていない場合にクラスターを自動的にパーティション化する方法の数または割合。
  #
  # デフォルト: 25%
  autoPartitionSize: 10%

  # パーティションの定義のリスト。ターゲットクラスターが構成に一致しない場合、autoPartitionSizeに従ってパーティションの最後に追加されます。
  partitions:

    # 表示用にパーティションに与えられるユーザーフレンドリーな名前（オプション）。
    # デフォルト: ""
    - name: canary

      # このパーティションで利用できないクラスターの数または割合。このパーティションが完了と見なされる前に利用できないクラスターの数または割合。
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

# ターゲットカスタマイズは、ターゲットごとにリソースをどのように変更するかを決定するために使用されます。ターゲットは順番に評価され、最初にクラスターに一致するものがそのクラスターに使用されます。
targetCustomizations:

  # ターゲットの名前。指定されていない場合、デフォルトの名前"target000"が使用されます。この値は主に表示用です。
  - name: prod

    # ルートの値を上書きするカスタムネームスペース値。
    namespace: newvalue

    # ルートの値を上書きするカスタムdefaultNamespace値。
    defaultNamespace: newdefaultvalue

    # ルートのオプションを上書きするカスタムkustomizeオプション。
    kustomize: {}

    # ルートのオプションを上書きするカスタムHelmオプション。
    helm: {}

    # 生のYAMLを使用する場合、リソースを置き換えまたはパッチするために使用されるoverlays/{name}にマップされる名前。./subdir/resource.yamlファイルをカスタマイズする場合、./overlays/myoverlay/subdir/resource.yamlファイルがベースファイルを置き換えます。./overlays/myoverlay/subdir/resource_patch.yamlファイルはベースファイルをパッチします。パッチはJSON Patch、JSON Merge形式、または組み込みのKubernetesタイプの戦略的マージパッチのいずれかである可能性があります。詳細については、以下の"Raw YAML Resource Customization"を参照してください。
    yaml:
      overlays:
        - custom2
        - custom3

    # クラスターを一致させるために使用されるセレクター。構造は標準のmetav1.LabelSelector形式です。clusterGroupSelectorまたはclusterGroupが指定されている場合、clusterSelectorはclusterGroupSelectorおよびclusterGroupが評価された後に選択をさらに絞り込むためにのみ使用されます。
    clusterSelector:
      matchLabels:
        env: prod

    # 名前で特定のクラスターに一致させるために使用されるセレクター。FleetをRancherで使用する場合、clusters.fleet.cattle.ioリソースの名前を必ず入力してください。
    clusterName: dev-cluster

    # クラスターグループに一致させるために使用されるセレクター。
    clusterGroupSelector:
      matchLabels:
        region: us-east

    # 選択される特定のclusterGroupの名前。
    clusterGroup: group1

    # doNotDeployがtrueの場合、一致するクラスターにリソースはデプロイされません。
    doNotDeploy: false

    # ドリフト修正は、Fleetによって管理されるリソースに対して行われた外部の変更を削除します。デフォルトでは三方向マージ戦略を使用するHelmのロールバックを実行します。forceが有効になっている場合、すべてのリソースをPUTリクエストで更新しようとします。三方向戦略的マージは、配列内のアイテムを更新しようとするときに失敗する可能性があります。これは、新しいアイテムを追加しようとするためです。
```
```markdown
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

これらのオプションは、kustomizeおよびmanifestスタイルのバンドルにも適用されます。これらは、fleet-agentがバンドルをデプロイする方法を制御します。すべてのバンドルはHelmチャートに変換され、Helm SDKを使用してデプロイされます。これらのオプションは、インストールおよび更新のためのHelm CLIオプションとよく似ています。

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

### テンプレート化

高度なテンプレート化のニーズに対応するために、キーと値をgoテンプレート文字列として指定することが可能です。ほとんどの関数は[sprigテンプレートライブラリ](https://masterminds.github.io/sprig/)から利用可能です。

関数の出力が毎回変わる場合、例えば`uuidv4`、バンドルは再デプロイされます。

テンプレートコンテキストには次のキーがあります:

* `.ClusterValues` はターゲットクラスターの `spec.templateValues` から取得されます
* `.ClusterLabels` および `.ClusterAnnotations` はクラスターリソースのラベルおよびアノテーションです。
* `.ClusterName` はfleetのクラスターリソース名です。
* `.ClusterNamespace` はクラスターリソースが存在するネームスペースです。

キー名でラベルやアノテーションにアクセスするには:

```
${ get .ClusterLabels "management.cattle.io/cluster-display-name" }
```

注意: fleet.yamlは有効なyamlでなければなりません。テンプレート化は`${ }`をデリミタとして使用しますが、Helmは`{{ }}`を使用します。これらのfleet.yamlテンプレートデリミタはバックティックでエスケープできます。例:

```
foo-bar-${`${PWD}`}
```

は次のテキストになります:

```
foo-bar-${PWD}
```
```