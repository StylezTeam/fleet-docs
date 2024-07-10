# カスタムリソース仕様

* [バンドル](#バンドル)
* [バンドルデプロイメント](#バンドルデプロイメント)
* [バンドルネームスペースマッピング](#バンドルネームスペースマッピング)
* [クラスター](#クラスター)
* [クラスターグループ](#クラスターグループ)
* [クラスター登録](#クラスター登録)
* [クラスター登録トークン](#クラスター登録トークン)
* [コンテンツ](#コンテンツ)
* [Gitリポジトリ](#gitリポジトリ)
* [Gitリポジトリ制限](#gitリポジトリ制限)
* [イメージスキャン](#イメージスキャン)

# サブリソース

* [バンドル表示](#バンドル表示)
* [バンドルリスト](#バンドルリスト)
* [バンドル参照](#バンドル参照)
* [バンドルリソース](#バンドルリソース)
* [バンドル仕様](#バンドル仕様)
* [バンドルステータス](#バンドルステータス)
* [バンドルサマリー](#バンドルサマリー)
* [バンドルターゲット](#バンドルターゲット)
* [バンドルターゲット制限](#バンドルターゲット制限)
* [非準備リソース](#非準備リソース)
* [パーティション](#パーティション)
* [パーティションステータス](#パーティションステータス)
* [リソースキー](#リソースキー)
* [ロールアウト戦略](#ロールアウト戦略)
* [バンドルデプロイメント表示](#バンドルデプロイメント表示)
* [バンドルデプロイメントリスト](#バンドルデプロイメントリスト)
* [バンドルデプロイメントオプション](#バンドルデプロイメントオプション)
* [バンドルデプロイメントリソース](#バンドルデプロイメントリソース)
* [バンドルデプロイメント仕様](#バンドルデプロイメント仕様)
* [バンドルデプロイメントステータス](#バンドルデプロイメントステータス)
* [比較パッチ](#比較パッチ)
* [ConfigMapキーセレクター](#configmapキーセレクター)
* [差分オプション](#差分オプション)
* [Helmオプション](#helmオプション)
* [無視オプション](#無視オプション)
* [Kustomizeオプション](#kustomizeオプション)
* [ローカルオブジェクト参照](#ローカルオブジェクト参照)
* [変更ステータス](#変更ステータス)
* [非準備ステータス](#非準備ステータス)
* [操作](#操作)
* [シークレットキーセレクター](#シークレットキーセレクター)
* [ValuesFrom](#valuesfrom)
* [YAMLオプション](#yamlオプション)
* [バンドルネームスペースマッピングリスト](#バンドルネームスペースマッピングリスト)
* [エージェントステータス](#エージェントステータス)
* [クラスター表示](#クラスター表示)
* [クラスターリスト](#クラスターリスト)
* [クラスター仕様](#クラスター仕様)
* [クラスターステータス](#クラスターステータス)
* [クラスターグループ表示](#クラスターグループ表示)
* [クラスターグループリスト](#クラスターグループリスト)
* [クラスターグループ仕様](#クラスターグループ仕様)
* [クラスターグループステータス](#クラスターグループステータス)
* [クラスター登録リスト](#クラスター登録リスト)
* [クラスター登録仕様](#クラスター登録仕様)
* [クラスター登録ステータス](#クラスター登録ステータス)
* [クラスター登録トークンリスト](#クラスター登録トークンリスト)
* [クラスター登録トークン仕様](#クラスター登録トークン仕様)
* [クラスター登録トークンステータス](#クラスター登録トークンステータス)
* [コンテンツリスト](#コンテンツリスト)
* [コミット仕様](#コミット仕様)
* [ドリフト修正](#ドリフト修正)
* [Gitリポジトリ表示](#gitリポジトリ表示)
* [Gitリポジトリリスト](#gitリポジトリリスト)
* [Gitリポジトリリソース](#gitリポジトリリソース)
* [Gitリポジトリリソースカウント](#gitリポジトリリソースカウント)
* [Gitリポジトリ仕様](#gitリポジトリ仕様)
* [Gitリポジトリステータス](#gitリポジトリステータス)
* [Gitターゲット](#gitターゲット)
* [クラスターごとのリソース状態](#クラスターごとのリソース状態)
* [Gitリポジトリ制限リスト](#gitリポジトリ制限リスト)
* [アルファベット順ポリシー](#アルファベット順ポリシー)
* [イメージポリシー選択](#イメージポリシー選択)
* [イメージスキャンリスト](#イメージスキャンリスト)
* [イメージスキャン仕様](#イメージスキャン仕様)
* [イメージスキャンステータス](#イメージスキャンステータス)
* [SemVerポリシー](#semverポリシー)

#### バンドル

バンドルはアプリケーションのリソースとそのデプロイメントオプションを含みます。ターゲットクラスターにHelmチャートとしてデプロイされます。

Gitリポジトリがスキャンされると、1つ以上のバンドルが生成されます。バンドルは1つ以上のクラスターにデプロイされるリソースのコレクションです。バンドルはFleetで使用される基本的なデプロイメント単位です。バンドルの内容はKubernetesマニフェスト、Kustomize構成、またはHelmチャートである可能性があります。ソースに関係なく、内容はエージェントによって動的にHelmチャートにレンダリングされ、下流のクラスターにHelmリリースとしてインストールされます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| spec |  | [バンドル仕様](#バンドル仕様) | true |
| status |  | [バンドルステータス](#バンドルステータス) | true |

[カスタムリソースに戻る](#カスタムリソース)

#### バンドル表示

バンドル表示には、準備ができているクラスター数、望ましい準備ができているクラスター数、およびバンドルのサマリーステートが含まれます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| readyClusters | ReadyClustersは「%d/%d」の形式の文字列で、準備ができているクラスター数と望ましい準備ができているクラスター数を示します。 | string | false |
| state | Stateは、準備ができていないリソースに基づいて計算されたバンドルのサマリーステートです。 | string | false |

[カスタムリソースに戻る](#カスタムリソース)

#### バンドルリスト

バンドルリストにはバンドルのリストが含まれます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][バンドル](#バンドル) | true |

[カスタムリソースに戻る](#カスタムリソース)

#### バンドル参照

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name | バンドルの名前。 | string | false |
| selector | バンドルのラベルに一致するセレクター。 | *metav1.LabelSelector | false |

[カスタムリソースに戻る](#カスタムリソース)

#### バンドルリソース

バンドルリソースは、バンドルからの単一リソースの内容を表します。例えば、YAMLマニフェストなどです。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name | リソースの名前。バンドルの内部パスを含むことができます。 | string | false |
| content | リソースの内容。圧縮されることがあります。 | string | false |
| encoding | エンコーディングは空または「base64+gz」です。 | string | false |

[カスタムリソースに戻る](#カスタムリソース)

#### バンドル仕様

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| paused | Pausedがtrueに設定されている場合、バンドルデプロイメントの更新が停止されます。同期が取れていないとマークされます。 | bool | false |
| rolloutStrategy | RolloutStrategyは、パーティション、カナリア、およびクラスターの可用性の割合を定義することによって、バンドルのロールアウトを制御します。 | *[ロールアウト戦略](#ロールアウト戦略) | false |
| resources | Resourcesには、バンドルのパスから読み取られたリソースが含まれます。これには、ダウンロードされたHelmチャートの内容が含まれます。 | \[\][バンドルリソース](#バンドルリソース) | false |
| targets | Targetsはデプロイされるクラスターを指します。ターゲットは順番に評価され、最初に一致するものが使用されます。 | \[\][バンドルターゲット](#バンドルターゲット) | false |
| targetRestrictions | TargetRestrictionsは許可リストであり、ターゲットに対してバンドルデプロイメントが作成されるかどうかを制御します。 | \[\][バンドルターゲット制限](#バンドルターゲット制限) | false |
| dependsOn | DependsOnは、このバンドルがデプロイされる前に準備ができている必要があるバンドルを指します。 | \[\][バンドル参照](#バンドル参照) | false |

[カスタムリソースに戻る](#カスタムリソース)

#### バンドルステータス

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| conditions | Conditionsは、バンドルの状態を説明するWrangler条件のリストです。 | []genericcondition.GenericCondition | false |
| summary | Summaryには、各状態のバンドルデプロイメントの数と準備ができていないリソースのリストが含まれます。 | [バンドルサマリー](#バンドルサマリー) | false |
| newlyCreated | NewlyCreatedは、作成されたが更新されていないバンドルデプロイメントの数です。 | int | false |
| unavailable | Unavailableは、準備ができていないか、ステータスのAppliedDeploymentIDが仕様のDeploymentIDと一致しないバンドルデプロイメントの数です。 | int | true |
| unavailablePartitions | UnavailablePartitionsは、利用できないパーティションの数です。 | int | true |
| maxUnavailable | MaxUnavailableは、利用できないデプロイメントの最大数です。ロールアウト構成を参照してください。 | int | true |
| maxUnavailablePartitions | MaxUnavailablePartitionsは、利用できないパーティションの最大数です。ロールアウト構成は、利用できないパーティションの最大数または割合を定義します。 | int | true |
| maxNew | MaxNewは常に50です。バンドルの変更は一度に50のバンドルデプロイメントしかステージングできません。 | int | false |
| partitions | PartitionStatusは各パーティションのステータスをリストします。 | \[\][パーティションステータス](#パーティションステータス) | false |
| display | Displayには、準備ができているクラスター数、望ましい準備ができているクラスター数、およびバンドルのリソースのサマリーステートが含まれます。 | [バンドル表示](#バンドル表示) | false |
| resourceKey | ResourceKeyは、デプロイされる可能性のあるリソースをリストします。クラスター上の実際のリソースリストは、Helmチャート、値のテンプレート化などに依存して異なる場合があります。 | \[\][リソースキー](#リソースキー) | false |
| observedGeneration | ObservedGenerationはバンドルの現在の世代です。 | int64 | true |
| resourcesSha256Sum | ResourcesSHA256Sumは、.Spec.ResourcesフィールドのJSONシリアル化に対応します。 | string | false |

[カスタムリソースに戻る](#カスタムリソース)

#### バンドルサマリー

バンドルサマリーには、各状態のバンドルデプロイメントの数と準備ができていないリソースのリストが含まれます。これはバンドル、クラスターグループ、クラスター、およびGitリポジトリのステータスで使用されます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| notReady | NotReadyは、いくつかのリソースが準備ができていない状態でデプロイされたバンドルデプロイメントの数です。 | int | false |
| waitApplied | WaitAppliedは、Fleetコントローラーと下流クラスターから同期されたが、デプロイを待っているバンドルデプロイメントの数です。 | int | false |
| errApplied | ErrAppliedは、Fleetコントローラーと下流クラスターから同期されたが、バンドルのデプロイ時にいくつかのエラーが発生したバンドルデプロイメントの数です。 | int | false |
| outOfSync | OutOfSyncは、Fleetコントローラーから同期されたが、まだ下流エージェントによって同期されていないバンドルデプロイメントの数です。 | int | false |
| modified | Modifiedは、すべてのリソースが準備ができている状態でデプロイされたが、Gitリポジトリからのいくつかの変更がまだ同期されていないバンドルデプロイメントの数です。 | int | false |
| ready | Readyは、すべてのリソースが準備ができている状態でデプロイされたバンドルデプロイメントの数です。 | int | true |
| pending | Pendingは、Fleetコントローラーによって処理されているバンドルデプロイメントの数です。 | int | false |
| desiredReady | DesiredReadyは、準備が整っているべきバンドルデプロイメントの数です。 | int | true |
| nonReadyResources | NonReadyClustersは、準備が整っていないバンドルの状態リストです。 | \[\][NonReadyResource](#nonreadyresource) | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleTarget

BundleTargetはデプロイするクラスターを宣言します。FleetはカスタマイズからのBundleDeploymentOptionsをこの構造体にマージします。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name | ターゲットの名前。この値は主に表示とログのためのものです。指定されない場合は、デフォルトの形式「target000」の名前が使用されます。 | string | false |
| clusterName | 特定のクラスターを名前で選択するためのClusterName。 | string | false |
| clusterSelector | ClusterSelectorはクラスターを一致させるためのセレクターです。この構造は標準のmetav1.LabelSelector形式です。clusterGroupSelectorまたはclusterGroupが指定されている場合、clusterSelectorはclusterGroupSelectorおよびclusterGroupの評価後に選択をさらに絞り込むために使用されます。 | *metav1.LabelSelector | false |
| clusterGroup | 特定のクラスターグループを名前で一致させるためのClusterGroup。 | string | false |
| clusterGroupSelector | ClusterGroupSelectorはクラスターグループを一致させるためのセレクターです。 | *metav1.LabelSelector | false |
| doNotDeploy | DoNotDeployがtrueに設定されている場合、このターゲットにデプロイしません。 | bool | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleTargetRestriction

BundleTargetRestrictionはFleetによって内部的に使用され、変更されるべきではありません。これは許可リストとして機能し、fleet.yamlでTargetCustomizationsによって作成されたターゲットからのBundleDeploymentsの作成を防ぎます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name |  | string | false |
| clusterName |  | string | false |
| clusterSelector |  | *metav1.LabelSelector | false |
| clusterGroup |  | string | false |
| clusterGroupSelector |  | *metav1.LabelSelector | false |

[カスタムリソースに戻る](#custom-resources)

#### NonReadyResource

NonReadyResourceは、特定の状態（例：「ErrApplied」）に対して準備が整っていないバンドルに関する情報を含みます。これは、準備が整っていないまたは変更されたリソースとその状態のリストを含みます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name | リソースの名前です。 | string | false |
| bundleState | リソースの状態（例：「NotReady」または「ErrApplied」）です。 | BundleState | false |
| message | バンドルが準備が整っていない理由を含むメッセージです。 | string | false |
| modifiedStatus | 変更された各リソースの状態をリストします。 | \[\][ModifiedStatus](#modifiedstatus) | false |
| nonReadyStatus | 準備が整っていない各リソースの状態をリストします。 | \[\][NonReadyStatus](#nonreadystatus) | false |

[カスタムリソースに戻る](#custom-resources)

#### Partition

Partitionは、クラスターのセットに対する別個のロールアウト戦略を定義します。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name | 表示用に与えられたユーザーフレンドリーな名前（オプション）。 | string | false |
| maxUnavailable | このパーティションで利用できないクラスターの数または割合。デフォルト: 10% | *intstr.IntOrString | false |
| clusterName | このパーティションに含めるクラスターの名前。 | string | false |
| clusterSelector | このパーティションに含めるクラスターラベルを一致させるセレクター。 | *metav1.LabelSelector | false |
| clusterGroup | このパーティションに含めるクラスターグループの名前。 | string | false |
| clusterGroupSelector | このパーティションに含めるクラスターグループラベルを一致させるセレクター。 | *metav1.LabelSelector | false |

[カスタムリソースに戻る](#custom-resources)

#### PartitionStatus

PartitionStatusは、単一のロールアウトパーティションのステータスです。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name | パーティションの名前です。 | string | false |
| count | パーティション内のクラスターの数です。 | int | false |
| maxUnavailable | パーティション内で利用できないクラスターの最大数です。 | int | false |
| unavailable | パーティション内で利用できないクラスターの数です。 | int | false |
| summary | パーティションの要約状態で、準備が整っていないリソースに基づいて計算されます。 | [BundleSummary](#bundlesummary) | false |

[カスタムリソースに戻る](#custom-resources)

#### ResourceKey

ResourceKeyは、デプロイされる可能性のあるリソースをリストします。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| kind | リソースのk8s APIの種類です。 | string | false |
| apiVersion | リソースのk8s APIバージョンです。 | string | false |
| namespace | リソースのネームスペースです。 | string | false |
| name | リソースの名前です。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### RolloutStrategy

RolloverStrategyは、クラスター全体でのバンドルのロールアウトを制御します。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| maxUnavailable | バンドルの更新中に利用できないクラスターの数または割合。このアプローチはデプロイメントのロールアウト戦略と同じ基本的なアプローチに従います。クラスターの数が利用できない状態に達すると、更新は一時停止されます。デフォルト値は100%で、更新には影響しません。デフォルト: 100% | *intstr.IntOrString | false |
| maxUnavailablePartitions | バンドルの更新中に利用できないクラスターのパーティションの数または割合。デフォルト: 0 | *intstr.IntOrString | false |
| autoPartitionSize | 特定のパーティショニング戦略が構成されていない場合にクラスターを自動的にパーティション化する方法の数または割合。デフォルト: 25% | *intstr.IntOrString | false |
| partitions | パーティションの定義のリスト。ターゲットクラスターが構成に一致しない場合、それらはautoPartitionSizeに従って最後にパーティションに追加されます。 | \[\][Partition](#partition) | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleDeployment

BundleDeploymentはFleetによって内部的に使用され、直接使用されるべきではありません。バンドルがクラスターにデプロイされると、バンドルのインスタンスはBundleDeploymentと呼ばれます。BundleDeploymentは、特定のクラスター上のバンドルの状態を、そのクラスター固有のカスタマイズと共に表します。Fleetエージェントは、エージェントが管理するクラスターのために作成されたBundleDeploymentリソースのみを認識します。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| spec |  | [BundleDeploymentSpec](#bundledeploymentspec) | false |
| status |  | [BundleDeploymentStatus](#bundledeploymentstatus) | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleDeploymentDisplay

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| deployed |  | string | false |
| monitored |  | string | false |
| state |  | string | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleDeploymentList

BundleDeploymentListは、BundleDeploymentのリストを含みます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][BundleDeployment](#bundledeployment) | true |

[カスタムリソースに戻る](#custom-resources)

#### BundleDeploymentOptions

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| defaultNamespace | DefaultNamespaceは、ネームスペースを指定しないリソースに使用するネームスペースです。このフィールドは、特定のネームスペースへのデプロイを強制またはロックダウンするために使用されません。 | string | false |
| namespace | TargetNamespaceが存在する場合、すべてのリソースをこのネームスペースに割り当て、クラスター範囲のリソースが存在する場合、デプロイは失敗します。 | string | false |
| kustomize | デプロイのためのKustomizeオプション（例：kustomization.yamlファイルを含むディレクトリ）。 | *[KustomizeOptions](#kustomizeoptions) | false |
| helm | デプロイのためのHelmオプション（例：チャート名、リポジトリ、値）。 | *[HelmOptions](#helmoptions) | false |
| serviceAccount | このデプロイを実行するために使用されるServiceAccount。 | string | false |
| forceSyncGeneration | ForceSyncGenerationは再デプロイを強制するために使用されます。 | int64 | false |
| yaml | YAMLオプション。生のYAMLを使用する場合、これらはリソースを置き換えまたはパッチするために使用されるoverlays/{name}ファイルにマップされる名前です。 | *[YAMLOptions](#yamloptions) | false |
| diff | Diffは、実行時に修正されたオブジェクトの変更状態を無視するために使用できます。 | *[DiffOptions](#diffoptions) | false |
| keepResources | バンドルを削除する際にデプロイされたリソースを保持するために使用できます。 | bool | false |
| ignore | バンドルを監視する際にフィールドを無視するためのIgnoreOptions。 | [IgnoreOptions](#ignoreoptions) | false |
| correctDrift | ドリフト修正の方法を指定します。 | *[CorrectDrift](#correctdrift) | false |
| namespaceLabels | Fleetによって作成されたネームスペースに追加されるラベル。 | *map[string]string | false |
| namespaceAnnotations | Fleetによって作成されたネームスペースに追加されるアノテーション。 | *map[string]string | false |
| deleteCRDResources | CRDを削除します。警告！これにより、すべてのカスタムリソースも削除されます。 | bool | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleDeploymentResource

BundleDeploymentResourceは、デプロイされたリソースのメタデータを含みます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| kind |  | string | false |
| apiVersion |  | string | false |
| namespace |  | string | false |
| name |  | string | false |
| createdAt |  | metav1.Time | false |
[カスタムリソースに戻る](#custom-resources)

#### BundleDeploymentSpec

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| paused | trueに設定されている場合、BundleDeploymentsの更新が停止されます。変更が検出された場合、BundleDeploymentsは同期外とマークされます。 | bool | false |
| stagedOptions | StagedOptionsは、次のデプロイメントのためにステージングされたデプロイメントオプションです。 | [BundleDeploymentOptions](#bundledeploymentoptions) | false |
| stagedDeploymentID | StagedDeploymentIDは、ステージングされたデプロイメントのIDです。 | string | false |
| options | Optionsは、現在適用されているデプロイメントオプションです。 | [BundleDeploymentOptions](#bundledeploymentoptions) | false |
| deploymentID | DeploymentIDは、現在適用されているデプロイメントのIDです。 | string | false |
| dependsOn | DependsOnは、このバンドルがデプロイされる前に準備が必要なバンドルを指します。 | \[\][BundleRef](#bundleref) | false |
| correctDrift | CorrectDriftは、ドリフト修正の方法を指定します。 | *[CorrectDrift](#correctdrift) | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleDeploymentStatus

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| conditions |  | []genericcondition.GenericCondition | false |
| appliedDeploymentID |  | string | false |
| release |  | string | false |
| ready |  | bool | false |
| nonModified |  | bool | false |
| nonReadyStatus |  | \[\][NonReadyStatus](#nonreadystatus) | false |
| modifiedStatus |  | \[\][ModifiedStatus](#modifiedstatus) | false |
| display |  | [BundleDeploymentDisplay](#bundledeploymentdisplay) | false |
| syncGeneration |  | *int64 | false |
| resources | Resourcesは、helmリリース履歴に従ってデプロイされたリソースのメタデータを一覧表示します。 | \[\][BundleDeploymentResource](#bundledeploymentresource) | false |

[カスタムリソースに戻る](#custom-resources)

#### ComparePatch

ComparePatchはリソースに一致し、変更のチェックからフィールドを削除します。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| kind | Kindは一致するリソースの種類です。 | string | false |
| apiVersion | APIVersionは一致するリソースのapiVersionです。 | string | false |
| namespace | Namespaceは一致するリソースの名前空間です。 | string | false |
| name | Nameは一致するリソースの名前です。 | string | false |
| operations | OperationsはリソースからJSONパスを削除します。 | \[\][Operation](#operation) | false |
| jsonPointers | JSONPointersは特定のJSONパスでの差分を無視します。 | []string | false |

[カスタムリソースに戻る](#custom-resources)

#### ConfigMapKeySelector

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| namespace |  | string | false |
| key |  | string | false |

[カスタムリソースに戻る](#custom-resources)

#### DiffOptions

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| comparePatches | ComparePatchesはリソースに一致し、変更のチェックからフィールドを削除します。 | \[\][ComparePatch](#comparepatch) | false |

[カスタムリソースに戻る](#custom-resources)

#### HelmOptions

HelmOptionsはデプロイメントのためのオプションです。Helmベースのバンドルの場合、すべてのオプションが使用できますが、その他のバンドルでは一部のオプションが無視されます。例えば、ReleaseNameはすべてのバンドルタイプで機能します。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| chart | Chartは任意のgo-getter URLまたはOCIレジストリベースのhelmチャートURLを指すことができます。チャートはダウンロードされます。 | string | false |
| repo | RepoはチャートをダウンロードするためのHTTPS helmリポジトリの名前です。 | string | false |
| releaseName | ReleaseNameはチャートをデプロイするためのカスタムリリース名を設定します。指定されていない場合、リリース名は呼び出し元のGitRepo.name + GitRepo.pathを組み合わせて生成されます。 | string | false |
| version | ダウンロードするチャートのバージョン | string | false |
| timeoutSeconds | TimeoutSecondsはHelm操作を待機する時間です。 | int | false |
| values | Helmに渡される値。キーと値をgoテンプレート文字列として指定することが可能です。 | *GenericMap | false |
| valuesFrom | ValuesFromはconfigmapsおよびsecretsから値をロードします。 | \[\][ValuesFrom](#valuesfrom) | false |
| force | Forceは不変のリソースを上書きすることを許可します。これは危険です。 | bool | false |
| takeOwnership | TakeOwnershipはHelmが自身のアノテーションのチェックをスキップするようにします。 | bool | false |
| maxHistory | MaxHistoryはHelmによって保存されるリリースごとの最大リビジョン数を制限します。 | int | false |
| valuesFiles | ValuesFilesは値をロードするファイルのリストです。 | []string | false |
| waitForJobs | WaitForJobsが設定され、timeoutSecondsが提供されている場合、すべてのジョブが完了するまで待機し、GitRepoを準備完了とマークします。timeoutSecondsの間待機します。 | bool | false |
| atomic | AtomicはHelmがアップグレードを実行する際に--atomicフラグを設定します。 | bool | false |
| disablePreProcess | DisablePreProcessは値のテンプレート処理を無効にします。 | bool | false |
| disableDNS | DisableDNSはHelmのEnableDNSオプションをカスタマイズするために使用できます。Fleetはデフォルトでこれを`true`に設定します。 | bool | false |
| skipSchemaValidation | SkipSchemaValidationはチャート値に対するスキーマ検証をスキップすることを許可します。 | bool | false |

[カスタムリソースに戻る](#custom-resources)

#### IgnoreOptions

IgnoreOptionsは、バンドルを監視する際に無視する条件を定義します。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| conditions | Conditionsは、バンドルを監視する際に無視する条件のリストです。 | []map[string]string | false |

[カスタムリソースに戻る](#custom-resources)

#### KustomizeOptions

KustomizeOptionsはデプロイメントのためのオプションです。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| dir | Dirはkustomizeリソースのためのカスタムフォルダを指します。このフォルダにはkustomization.yamlファイルが含まれている必要があります。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### LocalObjectReference

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name | 参照先と同じ名前空間内のリソースの名前。 | string | true |

[カスタムリソースに戻る](#custom-resources)

#### ModifiedStatus

ModifiedStatusは、リソースが変更された状態を報告するために使用されます。変更が作成、削除、またはパッチであったかを示します。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| kind |  | string | false |
| apiVersion |  | string | false |
| namespace |  | string | false |
| name |  | string | false |
| missing |  | bool | false |
| delete |  | bool | false |
| patch |  | string | false |

[カスタムリソースに戻る](#custom-resources)

#### NonReadyStatus

NonReadyStatusは、準備ができていないリソースの状態を報告するために使用されます。概要を含みます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| uid |  | types.UID | false |
| kind |  | string | false |
| apiVersion |  | string | false |
| namespace |  | string | false |
| name |  | string | false |
| summary |  | summary.Summary | false |

[カスタムリソースに戻る](#custom-resources)

#### Operation

OperationはComparePatchの操作で、通常は「remove」です。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| op | Opは通常「remove」です。 | string | false |
| path | Pathは削除するJSONパスです。 | string | false |
| value | Valueは通常空です。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### SecretKeySelector

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| namespace |  | string | false |
| key |  | string | false |

[カスタムリソースに戻る](#custom-resources)

#### ValuesFrom

configmap、secret、または外部から取得できるhelm値を定義します。クレジット: https://github.com/fluxcd/helm-operator/blob/0cfea875b5d44bea995abe7324819432070dfbdc/pkg/apis/helm.fluxcd.io/v1/types_helmrelease.go#L439

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| configMapKeyRef | リリース値を持つconfig mapへの参照。 | *[ConfigMapKeySelector](#configmapkeyselector) | false |
| secretKeyRef | リリース値を持つsecretへの参照。 | *[SecretKeySelector](#secretkeyselector) | false |

[カスタムリソースに戻る](#custom-resources)

#### YAMLOptions

YAMLOptionsは、生のYAMLを使用する場合に、リソースを置き換えたりパッチを適用するためにoverlays/{name}ファイルにマッピングされる名前です。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| overlays | Overlaysは「overlays/」フォルダ内のフォルダにマッピングされる名前のリストです。./subdir/resource.yamlファイルをカスタマイズしたい場合、./overlays/myoverlay/subdir/resource.yamlファイルがベースファイルを置き換えます。./overlays/myoverlay/subdir/resource_patch.yamlファイルはベースファイルにパッチを適用します。 | []string | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleNamespaceMapping

BundleNamespaceMappingは、バンドルを他の名前空間のクラスターにマッピングします。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| bundleSelector |  | *metav1.LabelSelector | false |
| namespaceSelector |  | *metav1.LabelSelector | false |

[カスタムリソースに戻る](#custom-resources)

#### BundleNamespaceMappingList

BundleNamespaceMappingListは、BundleNamespaceMappingのリストを含みます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][BundleNamespaceMapping](#bundlenamespacemapping) | true |

[カスタムリソースに戻る](#custom-resources)

#### AgentStatus

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| lastSeen | LastSeenは、エージェントがクラスターリソースのステータスを更新するために最後にチェックインした時間です。 | metav1.Time | true |
| namespace | Namespaceはエージェントデプロイメントの名前空間です。例：「cattle-fleet-system」。 | string | true |
| nonReadyNodes | NonReadyNodesは準備ができていないノードの数です。 | int | true |
| readyNodes | ReadyNodesは準備ができているノードの数です。 | int | true |
| nonReadyNodeNames | NonReadyNodeには準備ができていないノードの名前が含まれます。このリストは最大3つの名前に制限されています。 | []string | true |
| readyNodeNames | ReadyNodesには準備ができているノードの名前が含まれます。このリストは最大3つの名前に制限されています。 | []string | true |

[カスタムリソースに戻る](#custom-resources)

#### クラスター

クラスターはKubernetesクラスターに対応します。Fleetはターゲットクラスターにバンドルをデプロイします。Fleetがマニフェストをデプロイするクラスターはダウンストリームクラスターと呼ばれます。単一クラスターの使用例では、FleetマネージャーKubernetesクラスターがマネージャーとダウンストリームクラスターの両方を兼ねます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| spec |  | [ClusterSpec](#clusterspec) | false |
| status |  | [ClusterStatus](#clusterstatus) | false |

[カスタムリソースに戻る](#custom-resources)

#### クラスター表示

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| readyBundles | ReadyBundlesは「%d/%d」の形式の文字列で、準備ができているバンドルの数と準備が必要なバンドルの数を示します。 | string | false |
| readyNodes | ReadyNodesは「%d/%d」の形式の文字列で、準備ができているノードの数と期待されるノードの数を示します。 | string | false |
| sampleNode | SampleNodeは準備ができているノードの1つの名前です。準備ができていない場合は、準備ができていないノードの名前です。 | string | false |
| state | クラスターの状態。バンドルの状態のいずれか、または「WaitCheckIn」。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### クラスターリスト

クラスターリストにはクラスターのリストが含まれます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][Cluster](#cluster) | true |

[カスタムリソースに戻る](#custom-resources)

#### クラスター仕様

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| paused | Pausedがtrueに設定されている場合、BundleDeploymentsの更新が停止されます。 | bool | false |
| clientID | ClientIDはクラスターを識別する一意の文字列です。事前に定義することも、クラスターをインポートする際に生成することもできます。 | string | false |
| kubeConfigSecret | KubeConfigSecretはダウンストリームクラスターのkubeconfigを含むシークレットの名前です。オプションでAPIServerURLとCAを含めて、fleet-controllerのconfigmapの値を上書きすることができます。 | string | false |
| kubeConfigSecretNamespace | KubeConfigSecretNamespaceはダウンストリームクラスターのkubeconfigを含むシークレットの名前空間です。設定されていない場合、シークレットはクラスターオブジェクトが存在する名前空間にあると見なされます。 | string | false |
| redeployAgentGeneration | RedeployAgentGenerationはエージェントの再デプロイを強制するために使用できます。 | int64 | false |
| agentEnvVars | AgentEnvVarsはエージェントデプロイメントに追加される追加の環境変数です。 | []corev1.EnvVar | false |
| agentNamespace | AgentNamespaceはデフォルトでシステム名前空間、例：cattle-fleet-system。 | string | false |
| privateRepoURL | PrivateRepoURLはイメージ名のプレフィックスであり、エージェントの設定からグローバルリポジトリURLを上書きします。 | string | false |
| templateValues | TemplateValuesはfleet.yamlの値テンプレートに送信されるクラスター固有の値のマッピングを定義します。 | *GenericMap | false |
| agentTolerations | AgentTolerationsはエージェントデプロイメントに追加される追加のTolerationsセットを定義します。 | []corev1.Toleration | false |
| agentAffinity | AgentAffinityはクラスターのエージェントデプロイメントのデフォルトのアフィニティを上書きします。この値がnilの場合、デフォルトのアフィニティが使用されます。 | *corev1.Affinity | false |
| agentResources | AgentResourcesはクラスターのエージェントデプロイメントのリソースを設定します。 | *corev1.ResourceRequirements | false |

[カスタムリソースに戻る](#custom-resources)

#### クラスター状態

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| conditions |  | []genericcondition.GenericCondition | false |
| namespace | Namespaceはクラスターの名前空間で、クラスターのサービスアカウントやバンドルデプロイメントを含みます。例：「cluster-fleet-local-cluster-294db1acfa77-d9ccf852678f」 | string | false |
| summary | Summaryはバンドルデプロイメントの概要です。リソースカウントはgitrepoリソースからコピーされます。 | [BundleSummary](#bundlesummary) | false |
| resourceCounts | ResourceCountsはGitRepoResourceCountsの集計です。 | [GitRepoResourceCounts](#gitreporesourcecounts) | false |
| readyGitRepos | ReadyGitReposはこのクラスターの準備ができているgitreposの数です。 | int | true |
| desiredReadyGitRepos | DesiredReadyGitReposはこのクラスターの準備が必要なgitreposの数です。 | int | true |
| agentEnvVarsHash | AgentEnvVarsHashはエージェントの環境変数のハッシュで、変更を検出するために使用されます。 | string | false |
| agentPrivateRepoURL | AgentPrivateRepoURLは現在使用されているエージェントのプライベートリポジトリURLです。 | string | false |
| agentDeployedGeneration | AgentDeployedGenerationは現在デプロイされているエージェントの世代です。 | *int64 | false |
| agentMigrated | AgentMigratedはクラスターをインポートした後に常にtrueに設定されます。falseの場合、マイグレーションがトリガーされます。古いエージェントにはこのステータスがありません。 | bool | false |
| agentNamespaceMigrated | AgentNamespaceMigratedはクラスターをインポートした後に常にtrueに設定されます。falseの場合、マイグレーションがトリガーされます。古いFleetエージェントにはこのステータスがありません。 | bool | false |
| cattleNamespaceMigrated | CattleNamespaceMigratedはクラスターをインポートした後に常にtrueに設定されます。falseの場合、マイグレーションがトリガーされます。古いFleetエージェントにはこのステータスがありません。 | bool | false |
| agentAffinityHash | AgentAffinityHashはエージェントのアフィニティ設定のハッシュで、変更を検出するために使用されます。 | string | false |
| agentResourcesHash | AgentResourcesHashはエージェントのリソース設定のハッシュで、変更を検出するために使用されます。 | string | false |
| agentTolerationsHash | AgentTolerationsHashはエージェントのTolerations設定のハッシュで、変更を検出するために使用されます。 | string | false |
| agentConfigChanged | AgentConfigChangedはエージェントの設定（APIサーバーURLやCAなど）が変更された場合にtrueに設定されます。trueに設定すると、クラスターの再インポートがトリガーされます。 | bool | false |
| apiServerURL | APIServerURLはクラスターがアップストリームに接続するために使用する現在のAPIサーバーのURLです。 | string | false |
| apiServerCAHash | APIServerCAHashはアップストリームAPIサーバーCAのハッシュで、変更を検出するために使用されます。 | string | false |
| display | Displayには準備ができているバンドル、ノードの数、および概要状態が含まれます。 | [ClusterDisplay](#clusterdisplay) | false |
| agent | AgentStatusにはエージェントに関する情報が含まれます。 | [AgentStatus](#agentstatus) | false |

[カスタムリソースに戻る](#custom-resources)

#### クラスターグループ

クラスターグループは、クラスターのグループをターゲットにするための再利用可能なセレクターです。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| spec |  | [ClusterGroupSpec](#clustergroupspec) | true |
| status |  | [ClusterGroupStatus](#clustergroupstatus) | true |

[カスタムリソースに戻る](#custom-resources)

#### クラスターグループ表示

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| readyClusters | ReadyClustersは「%d/%d」の形式の文字列で、準備ができているクラスターの数と準備が必要なクラスターの数を示します。 | string | false |
| readyBundles | ReadyBundlesは「%d/%d」の形式の文字列で、準備ができているバンドルの数と準備が必要なバンドルの数を示します。 | string | false |
| state | Stateはクラスターグループの概要状態で、準備ができていないリソースがある場合は「NotReady」を示します。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### クラスターグループリスト

クラスターグループリストにはクラスターグループのリストが含まれます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][ClusterGroup](#clustergroup) | true |

[カスタムリソースに戻る](#custom-resources)

#### クラスターグループ仕様

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| selector | Selectorはラベルセレクターで、このグループのクラスターを選択するために使用されます。 | *metav1.LabelSelector | false |

[カスタムリソースに戻る](#custom-resources)

#### クラスターグループ状態

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| clusterCount | ClusterCountはクラスターグループ内のクラスターの数です。 | int | true |
| nonReadyClusterCount | NonReadyClusterCountは準備ができていないクラスターの数です。 | int | true |
| nonReadyClusters | NonReadyClustersは準備ができていないクラスターの名前のリストです。 | []string | false |
| conditions | Conditionsはクラスターグループの条件とそのステータスのリストです。 | []genericcondition.GenericCondition | false |
| summary | Summaryはクラスターグループ内のバンドルデプロイメントとそのリソースの概要です。 | [BundleSummary](#bundlesummary) | false |
| display | Displayには準備ができているクラスター、準備が必要なクラスターの数、およびバンドルのリソースの概要状態が含まれます。 | [ClusterGroupDisplay](#clustergroupdisplay) | false |
| resourceCounts | ResourceCountsにはクラスターグループ内のすべてのバンドルの各状態のリソースの数が含まれます。 | [GitRepoResourceCounts](#gitreporesourcecounts) | false |

[カスタムリソースに戻る](#custom-resources)

#### クラスター登録

クラスター登録はFleetによって内部的に使用され、直接使用されるべきではありません。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| spec |  | [ClusterRegistrationSpec](#clusterregistrationspec) | false |
| status |  | [ClusterRegistrationStatus](#clusterregistrationstatus) | false |

[カスタムリソースに戻る](#custom-resources)

#### ClusterRegistrationList

ClusterRegistrationListにはClusterRegistrationのリストが含まれています

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][ClusterRegistration](#clusterregistration) | true |

[カスタムリソースに戻る](#custom-resources)

#### ClusterRegistrationSpec

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| clientID | ClientIDはクラスターを識別する一意の文字列です。エージェントは設定されたIDまたはkubeSystem.UIDを使用します。 | string | false |
| clientRandom | ClientRandomはエージェントが生成するランダムな文字列です。fleet-controllerが登録を許可すると、この文字列を名前に含む登録シークレットを作成します。 | string | false |
| clusterLabels | ClusterLabelsは登録時にクラスターリソースにコピーされます。 | map[string]string | false |

[カスタムリソースに戻る](#custom-resources)

#### ClusterRegistrationStatus

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| clusterName | ClusterNameは、登録がfleet-controllerによって処理されているときにのみ設定されます。 | string | false |
| granted | Grantedは、リクエストサービスアカウントが存在し、そのトークンシークレットが存在する場合にtrueに設定されます。これは、登録シークレット、ロール、およびロールバインディングを作成する直前に発生します。 | bool | false |

[カスタムリソースに戻る](#custom-resources)

#### ClusterRegistrationToken

ClusterRegistrationTokenは、エージェントが新しいクラスターを登録するために使用されます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| spec |  | [ClusterRegistrationTokenSpec](#clusterregistrationtokenspec) | false |
| status |  | [ClusterRegistrationTokenStatus](#clusterregistrationtokenstatus) | false |

[カスタムリソースに戻る](#custom-resources)

#### ClusterRegistrationTokenList

ClusterRegistrationTokenListにはClusterRegistrationTokenのリストが含まれています

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][ClusterRegistrationToken](#clusterregistrationtoken) | true |

[カスタムリソースに戻る](#custom-resources)

#### ClusterRegistrationTokenSpec

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| ttl | TTLはトークンの有効期限です。これを使用して有効期限を計算します。トークンが期限切れになると削除されます。 | *metav1.Duration | false |

[カスタムリソースに戻る](#custom-resources)

#### ClusterRegistrationTokenStatus

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| expires | Expiresはトークンの有効期限です。 | *metav1.Time | false |
| secretName | SecretNameはトークンを含むシークレットの名前です。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### Content

ContentはFleetによって内部的に使用され、直接使用されるべきではありません。特定のターゲットクラスターのバンドルからのリソースが含まれています。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| content | Contentはバイト配列で、バンドルのマニフェストが含まれています。バンドルリソースはbundledeploymentのcontentリソースにコピーされ、下流のエージェントがそれらをデプロイできるようにします。 | []byte | false |
| sha256sum | ContentフィールドのSHA256Sum | string | false |

[カスタムリソースに戻る](#custom-resources)

#### ContentList

ContentListにはContentのリストが含まれています

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][Content](#content) | true |

[カスタムリソースに戻る](#custom-resources)

#### CommitSpec

CommitSpecは、gitリポジトリへの変更をコミットする方法を指定します

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| authorName | AuthorNameはコミット時に提供する名前を指定します | string | true |
| authorEmail | AuthorEmailはコミット時に提供するメールアドレスを指定します | string | true |
| messageTemplate | MessageTemplateは、行われた変更の詳細を挿入するためのコミットメッセージのテンプレートを提供します。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### CorrectDrift

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| enabled | trueの場合、ドリフト修正を有効にします。 | bool | false |
| force | trueの場合、--forceオプションを使用してhelmロールバックを強制します。これにより、リリース内のすべてのリソースを再作成しようとします。 | bool | false |
| keepFailHistory | keepFailHistoryは、helm履歴に失敗したロールバックを記録します。 | bool | false |

[カスタムリソースに戻る](#custom-resources)

#### GitRepo

GitRepoは、Fleetによって監視されるgitリポジトリを記述します。このリソースには、リポジトリまたはその一部をターゲットクラスターにデプロイするために必要な情報が含まれています。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| spec |  | [GitRepoSpec](#gitrepospec) | false |
| status |  | [GitRepoStatus](#gitrepostatus) | false |

[カスタムリソースに戻る](#custom-resources)

#### GitRepoDisplay

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| readyBundleDeployments | ReadyBundleDeploymentsは「%d/%d」の形式の文字列で、準備ができているbundledeploymentsの数を総bundledeployments数に対して示します。 | string | false |
| state | StateはGitRepoの状態です。例：「GitUpdating」またはStateRankに従った最大のBundleState。 | string | false |
| message | Messageはデプロイメント条件からの関連メッセージを含みます。 | string | false |
| error | メッセージが存在する場合、Errorはtrueです。 | bool | false |

[カスタムリソースに戻る](#custom-resources)

#### GitRepoList

GitRepoListにはGitRepoのリストが含まれています

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][GitRepo](#gitrepo) | true |

[カスタムリソースに戻る](#custom-resources)

#### GitRepoResource

GitRepoResourceにはバンドルのリソースに関するメタデータが含まれています。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| apiVersion | APIVersionはリソースのAPIバージョンです。 | string | false |
| kind | Kindはリソースのk8s種類です。 | string | false |
| type | Typeはリソースの種類です。例：「apiextensions.k8s.io.customresourcedefinition」または「configmap」。 | string | false |
| id | IDはリソースの名前です。例：「namespace1/my-config」または「backingimagemanagers.storage.io」。 | string | false |
| namespace | リソースの名前空間。 | string | false |
| name | リソースの名前。 | string | false |
| incompleteState | バンドルサマリーに10個以上の準備ができていないリソースがあるか、準備ができていないリソースに10個以上の準備ができていないまたは変更された状態がある場合、IncompleteStateはtrueです。 | bool | false |
| state | Stateはリソースの状態です。例：「Unknown」、「WaitApplied」、「ErrApplied」または「Ready」。 | string | false |
| error | PerClusterStateのいずれかにエラーがある場合、Errorはtrueです。 | bool | false |
| transitioning | PerClusterStateのいずれかが移行中の場合、Transitioningはtrueです。 | bool | false |
| message | PerClusterStatesからの最初のメッセージ。 | string | false |
| perClusterState | PerClusterStateは各クラスターの状態のリストです。サマリーの準備ができていないリソースから派生。 | \[\][ResourcePerClusterState](#resourceperclusterstate) | false |

[カスタムリソースに戻る](#custom-resources)

#### GitRepoResourceCounts

GitRepoResourceCountsには各状態のリソース数が含まれています。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| ready | Readyは準備ができているリソースの数です。 | int | true |
| desiredReady | DesiredReadyは準備ができているべきリソースの数です。 | int | true |
| waitApplied | WaitAppliedは適用を待っているリソースの数です。 | int | true |
| modified | Modifiedは変更されたリソースの数です。 | int | true |
| orphaned | Orphanedは孤立したリソースの数です。 | int | true |
| missing | Missingは欠落しているリソースの数です。 | int | true |
| unknown | Unknownは不明な状態のリソースの数です。 | int | true |
| notReady | NotReadyは準備ができていないリソースの数です。リソースが他の状態に一致しない場合、準備ができていません。 | int | true |

[カスタムリソースに戻る](#custom-resources)

#### GitRepoSpec

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| repo | Repoはクローンしてインデックスを作成するgitリポジトリのURLです。 | string | false |
| branch | Branchは追跡するgitブランチです。 | string | false |
| revision | Revisionは操作する特定のコミットまたはタグです。 | string | false |
| targetNamespace | すべてのリソースがこの名前空間に作成されることを保証します。クラスター範囲のリソースは、これが設定されている場合に拒否されます。さらに、この名前空間はオンデマンドで作成されます。 | string | false |
| clientSecretName | ClientSecretNameはリポジトリに接続するために使用されるクライアントシークレットの名前です。シークレットは「kubernetes.io/basic-auth」または「kubernetes.io/ssh-auth」のタイプであることが期待されます。 | string | false |
| helmSecretName | HelmSecretNameはプライベートHelmリポジトリの認証シークレットを含みます。 | string | false |
| helmSecretNameForPaths | HelmSecretNameForPathsは各パスのプライベートHelmリポジトリの認証シークレットを含みます。 | string | false |
| helmRepoURLRegex | HelmRepoURLRegex Helmの認証情報は、Helmリポジトリがこの正規表現に一致する場合に使用されます。これが空または提供されていない場合、認証情報は常に使用されます。 | string | false |
| caBundle | CABundleは、リポジトリの証明書を検証するために使用されるPEMエンコードされたCAバンドルです。 | []byte | false |
| insecureSkipTLSVerify | InsecureSkipTLSverifyは、安全でないHTTPSを使用してリポジトリをクローンします。 | bool | false |
| paths | Pathsは、Gitリポジトリのルートから相対的なディレクトリで、適用するリソースを含むものです。パスのグロビングがサポートされており、例えば["charts/*"]はcharts/のサブディレクトリとしてすべてのフォルダに一致します。空の場合、"/"がデフォルトです。 | []string | false |
| paused | Pausedがtrueの場合、Gitの変更がクラスターに伝播されず、代わりにリソースがOutOfSyncとしてマークされます。 | bool | false |
| serviceAccount | デプロイメントのためにダウンストリームクラスターで使用されるServiceAccount。 | string | false |
| targets | Targetsは、このリポジトリがデプロイするターゲットのリストです。 | \[\][GitTarget](#gittarget) | false |
| pollingInterval | PollingIntervalは、Gitの新しい更新をチェックする頻度です。 | *metav1.Duration | false |
| forceSyncGeneration | Gitからのコンテンツの再デプロイを強制するためにこの番号をインクリメントします。 | int64 | false |
| imageScanInterval | ImageScanIntervalは、スキャンされたイメージを同期し、Gitリポジトリに書き戻す間隔です。 | *metav1.Duration | false |
| imageScanCommit | Commitは、新しいイメージがスキャンされてGitリポジトリに書き戻されたときに、Gitリポジトリにコミットする方法を指定します。 | [CommitSpec](#commitspec) | false |
| keepResources | KeepResourcesは、GitRepoを削除した後も作成されたリソースを保持するかどうかを指定します。 | bool | false |
| correctDrift | CorrectDriftは、ドリフト修正の方法を指定します。 | *[CorrectDrift](#correctdrift) | false |

[カスタムリソースに戻る](#custom-resources)

#### GitRepoStatus

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| observedGeneration | ObservedGenerationは、クラスター内のリソースの現在の世代です。これはk8sのmetadata.Generationからコピーされます。この値は、.metadataまたは.statusの変更を除くすべての変更に対してインクリメントされます。 | int64 | true |
| commit | Commitは、最後のgitjob実行からのGitコミットハッシュです。 | string | false |
| readyClusters | ReadyClustersは、このGitRepoのすべてのバンドルにわたって準備ができているクラスターの最小数です。 | int | true |
| desiredReadyClusters | DesiredReadyClustersは、このGitRepoのバンドルのために準備ができているべきクラスターの数です。 | int | true |
| gitJobStatus | GitJobStatusは、最後のGitJob実行のステータスです。エラーがなければ「Current」となります。 | string | false |
| summary | Summaryには、各状態のバンドルデプロイメントの数と準備ができていないリソースのリストが含まれます。 | [BundleSummary](#bundlesummary) | false |
| display | Displayには、ステータスの人間が読める要約が含まれます。 | [GitRepoDisplay](#gitrepodisplay) | false |
| conditions | Conditionsは、GitRepoの状態を説明するWrangler条件のリストです。 | []genericcondition.GenericCondition | false |
| resources | Resourcesには、各バンドルのリソースに関するメタデータが含まれます。 | \[\][GitRepoResource](#gitreporesource) | false |
| resourceCounts | ResourceCountsには、すべてのバンドルにわたる各状態のリソースの数が含まれます。 | [GitRepoResourceCounts](#gitreporesourcecounts) | false |
| resourceErrors | ResourceErrorsは、リソースからのエラーのソートされたリストです。 | []string | false |
| lastSyncedImageScanTime | LastSyncedImageScanTimeは、最後のイメージスキャンの時間です。 | metav1.Time | false |

[カスタムリソースに戻る](#custom-resources)

#### GitTarget

GitTargetは、デプロイするクラスターまたはクラスターグループです。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| name | Nameは、このターゲットの名前です。 | string | false |
| clusterName | ClusterNameはクラスターの名前です。 | string | false |
| clusterSelector | ClusterSelectorは、クラスターを選択するためのラベルセレクターです。 | *metav1.LabelSelector | false |
| clusterGroup | ClusterGroupは、クラスターと同じネームスペース内のクラスターグループの名前です。 | string | false |
| clusterGroupSelector | ClusterGroupSelectorは、クラスターグループを選択するためのラベルセレクターです。 | *metav1.LabelSelector | false |

[カスタムリソースに戻る](#custom-resources)

#### ResourcePerClusterState

ResourcePerClusterStateは、バンドルの準備ができていない各リソースに対して生成されます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| state | Stateはリソースの状態です。 | string | false |
| error | Errorは、リソースがエラー状態にある場合にtrueです。これは、準備ができていないリソースのバンドルの要約からコピーされます。 | bool | false |
| transitioning | Transitioningは、リソースが移行中の状態にある場合にtrueです。これは、準備ができていないリソースのバンドルの要約からコピーされます。 | bool | false |
| message | Messageは、バンドルの要約からのメッセージを結合します。メッセージはデリミタ「;」で結合されます。 | string | false |
| patch | 変更されたリソースのパッチ。 | *GenericMap | false |
| clusterId | ClusterIDはクラスターのIDです。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### GitRepoRestriction

GitRepoRestrictionは、同じネームスペース内のGitRepoのオプションを制限するためにオプションで使用できるリソースです。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| defaultServiceAccount | DefaultServiceAccountは、GitRepoのデフォルトのサービスアカウントを上書きします。 | string | false |
| allowedServiceAccounts | AllowedServiceAccountsは、GitRepoが使用できるサービスアカウントのリストです。 | []string | false |
| allowedRepoPatterns | AllowedRepoPatternsは、GitRepoのRepoフィールドの有効な値を制限する正規表現パターンのリストです。 | []string | false |
| defaultClientSecretName | DefaultClientSecretNameは、GitRepoのデフォルトのクライアントシークレットを上書きします。 | string | false |
| allowedClientSecretNames | AllowedClientSecretNamesは、GitRepoが使用できるクライアントシークレット名のリストです。 | []string | false |
| allowedTargetNamespaces | AllowedTargetNamespacesは、TargetNamespaceを指定されたネームスペースに制限します。AllowedTargetNamespacesが設定されている場合、TargetNamespaceを設定する必要があります。 | []string | false |

[カスタムリソースに戻る](#custom-resources)

#### GitRepoRestrictionList

GitRepoRestrictionListには、GitRepoRestrictionのリストが含まれます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][GitRepoRestriction](#gitreporestriction) | true |

[カスタムリソースに戻る](#custom-resources)

#### AlphabeticalPolicy

AlphabeticalPolicyは、アルファベット順の並べ替えポリシーを指定します。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| order | Orderは、タグの並べ替え順序を指定します。アルファベットの文字をタグとして、昇順はZを選択し、降順はAを選択します。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### ImagePolicyChoice

ImagePolicyChoiceは、提供できるすべてのポリシータイプのユニオンです。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| semver | SemVerは、利用可能なタグに対してチェックするセマンティックバージョン範囲を提供します。 | *[SemVerPolicy](#semverpolicy) | false |
| alphabetical | アルファベット順のタグの並べ替えルールセット。 | *[AlphabeticalPolicy](#alphabeticalpolicy) | false |

[カスタムリソースに戻る](#custom-resources)

#### ImageScan

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ObjectMeta | false |
| spec |  | [ImageScanSpec](#imagescanspec) | false |
| status |  | [ImageScanStatus](#imagescanstatus) | false |

[カスタムリソースに戻る](#custom-resources)

#### ImageScanList

ImageScanListには、ImageScanのリストが含まれます。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| metadata |  | metav1.ListMeta | false |
| items |  | \[\][ImageScan](#imagescan) | true |

[カスタムリソースに戻る](#custom-resources)

#### ImageScanSpec

APIはhttps://github.com/fluxcd/image-reflector-controllerから取得されています。

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| tagName | TagNameは、フィールドを置き換えるためにマニフェストに入れる必要があるタグ参照です。 | string | false |
| gitrepoName | GitRepoの参照名。 | string | false |
| image | Imageは、イメージリポジトリの名前です。 | string | false |
| interval | Intervalは、イメージリポジトリのスキャン間隔の長さです。 | metav1.Duration | false |
| secretRef | SecretRefは、イメージレジストリの認証情報を含むシークレットの名前を指定できます。シークレットは`kubectl create secret docker-registry`または同等のコマンドで作成する必要があります。 | *corev1.LocalObjectReference | false |
| suspend | このフラグは、コントローラーに後続のイメージスキャンを一時停止するように指示します。すでに開始されたスキャンには適用されません。デフォルトはfalseです。 | bool | false |
| policy | Policyは、最新のイメージを選択する際に従うべきポリシーの詳細を提供します。 | [ImagePolicyChoice](#imagepolicychoice) | true |

[カスタムリソースに戻る](#custom-resources)

#### ImageScanStatus

| フィールド | 説明 | スキーム | 必須 |
| ----- | ----------- | ------ | -------- |
| conditions |  | []genericcondition.GenericCondition | false |
| lastScanTime | LastScanTimeは、最後にイメージがスキャンされた時間です。 | metav1.Time | false |
| latestImage | LatestImageは、ポリシーに従ってフィルタリングおよび順序付けされたイメージリポジトリによってスキャンされたイメージのリストの最初のものを示します。 | string | false |
| latestTag | Latest tagは、ポリシーによってフィルタリングされた最新のタグです。 | string | false |
| latestDigest | LatestDigestは、最新のタグのダイジェストです。 | string | false |
| observedGeneration |  | int64 | false |
| canonicalImageName | CanonicalName は、すべての暗黙のビットを明示的にしたイメージリポジトリの名前です。例えば、`alpine` ではなく `docker.io/library/alpine` のようになります。 | string | false |

[カスタムリソースに戻る](#custom-resources)

#### SemVerPolicy

SemVerPolicy はセマンティックバージョンポリシーを指定します。

| フィールド | 説明 | スキーム | 必須 |
| --------- | ---- | -------- | ---- |
| range | Range はイメージタグのためのセムバーバージョン範囲を指定します。範囲内の最高バージョンのタグが最新のイメージを生成します。 | string | true |

[カスタムリソースに戻る](#custom-resources)