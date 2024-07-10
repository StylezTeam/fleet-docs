# 下流クラスターへのマッピング

[Fleet in Rancher](https://rancher.com/docs/rancher/v2.6/en/deploy-across-clusters/fleet/) は、ユーザーがクラスターを1つのクラスターのように簡単に管理できるようにします。ユーザーは、デプロイメントマニフェストやその他のKubernetesリソースで構成されるバンドルを、グループ化設定を使用してクラスター全体にデプロイできます。

:::info

__マルチクラスターのみ__:
このアプローチは、Fleetをマルチクラスター形式で実行している場合にのみ適用されます。
ターゲットが指定されていない場合、つまりシングルクラスターを使用している場合、バンドルはデフォルトのクラスターグループをターゲットにします。

:::

`GitRepos`を下流クラスターにデプロイする際には、クラスターをターゲットにマッピングする必要があります。

## ターゲットの定義

`GitRepo`のデプロイメントターゲットは、`spec.targets`フィールドを使用してクラスターまたはクラスターグループにマッチさせます。YAMLの仕様は以下の通りです。

```yaml
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: myrepo
  namespace: clusters
spec:
  repo: https://github.com/rancher/fleet-examples
  paths:
  - simple

  # ターゲットは順番に評価され、最初にマッチしたものが使用されます。
  # どのターゲットにもマッチしない場合、評価されたクラスターにはデプロイされません。
  targets:
  # ターゲットの名前。この値は主に表示とログのためのものです。
  # 指定されていない場合、デフォルトの形式 "target000" の名前が使用されます。
  - name: prod
    # クラスターをマッチさせるためのセレクター。構造は標準の
    # metav1.LabelSelector形式です。clusterGroupSelectorまたはclusterGroupが指定されている場合、
    # clusterSelectorはclusterGroupSelectorおよびclusterGroupが評価された後に選択をさらに絞り込むために使用されます。
    clusterSelector:
      matchLabels:
        env: prod
    # クラスターグループをマッチさせるためのセレクター。
    clusterGroupSelector:
      matchLabels:
        region: us-east
    # 名前で指定された特定のクラスターグループが選択されます。
    clusterGroup: group1
    # 名前で指定された特定のクラスターが選択されます。
    clusterName: cluster1
```

## ターゲットのマッチング

`GitRepo`と同じネームスペース内のすべてのクラスターおよびクラスターグループがすべてのターゲットに対して評価されます。
ターゲットのいずれかがクラスターにマッチした場合、`GitRepo`は下流クラスターにデプロイされます。
マッチしない場合、`GitRepo`はそのクラスターにデプロイされません。

クラスターをマッチさせるためのアプローチは3つあります。
クラスターセレクター、クラスターグループセレクター、または明示的なクラスターグループ名を使用できます。すべての基準は加算されるため、
最終的なマッチは "clusterSelector && clusterGroupSelector && clusterGroup" として評価されます。3つのうちのいずれかが
デフォルト値を持つ場合、それは基準から除外されます。デフォルト値はnullまたは""です。セレクターの値が`{}`であることは「すべてにマッチする」ことを意味することを理解することが重要です。

```yaml
targets:
  # すべてにマッチ
  - clusterSelector: {}
  # セレクター無視
  - clusterSelector: null
```

クラスター名でクラスターをマッチさせることもできます：

```yaml
targets:
  - clusterName: fleetname
```
RancherでFleetを使用する場合、`clusters.fleet.cattle.io`リソースの名前を必ず指定してください。

## デフォルトターゲット

`GitRepo`にターゲットが設定されていない場合、デフォルトのターゲット値が適用されます。デフォルトのターゲット値は以下の通りです。

```yaml
targets:
- name: default
  clusterGroup: default
```

これは、デフォルトの場所に設定されていないGitReposを設定したい場合、デフォルトと呼ばれるクラスターグループを作成し、
クラスターを追加するだけで済むことを意味します。

## クラスターごとのカスタマイズ

:::info

`GitRepo`リソースの`targets:`はデプロイするクラスターを選択します。`fleet.yaml`の`targetCustomizations:`はHelmの値を上書きするだけで、ターゲティングを変更しません。

:::

Fleetを使用してカスタマイズを行いながら異なるクラスターにKubernetesマニフェストをデプロイする方法を示すために、[multi-cluster/helm/fleet.yaml](https://github.com/rancher/fleet-examples/blob/master/multi-cluster/helm/fleet.yaml)を使用します。

**状況:** ユーザーは`env=dev`、`env=test`、`env=prod`の3つの異なるラベルを持つ3つのクラスターを持っています。ユーザーはこれらのクラスターにフロントエンドアプリケーションとバックエンドデータベースをデプロイしたいと考えています。

**期待される動作:**

- `dev`クラスターにデプロイした後、データベースのレプリケーションは有効になりません。
- `test`クラスターにデプロイした後、データベースのレプリケーションが有効になります。
- `prod`クラスターにデプロイした後、データベースのレプリケーションが有効になり、ロードバランサーサービスが公開されます。

**Fleetの利点:**

アプリを各クラスターにデプロイする代わりに、Fleetを使用すると以下の手順で全クラスターにデプロイできます：

1. gitRepo `https://github.com/rancher/fleet-examples.git` をデプロイし、パス `multi-cluster/helm` を指定します。
2. `multi-cluster/helm`の下で、Helmチャートがフロントエンドアプリケーションサービスとバックエンドデータベースサービスをデプロイします。
3. `fleet.yaml`に以下のルールが定義されます：

```
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

**結果:**

Fleetはカスタマイズされた`values.yaml`を使用してHelmチャートを異なるクラスターにデプロイします。

>**注:** 構成管理はデプロイメントに限定されず、一般的な構成管理にも拡張できます。Fleetは、任意のクラスターセット間でカスタマイズを通じて構成管理を自動的に適用できます。

### サポートされているカスタマイズ

* [DefaultNamespace](/ref-crds#bundledeploymentoptions)
* [ForceSyncGeneration](/ref-crds#bundledeploymentoptions)
* [KeepResources](/ref-crds#bundledeploymentoptions)
* [ServiceAccount](/ref-crds#bundledeploymentoptions)
* [TargetNamespace](/ref-crds#bundledeploymentoptions)
* [Helm.Atomic](/ref-crds#helmoptions)
* [Helm.Chart](/ref-crds#helmoptions)
* [Helm.DisablePreProcess](/ref-crds#helmoptions)
* [Helm.Force](/ref-crds#helmoptions)
* [Helm.ReleaseName](/ref-crds#helmoptions)
* [Helm.Repo](/ref-crds#helmoptions)
* [Helm.TakeOwnership](/ref-crds#helmoptions)
* [Helm.TimeoutSeconds](/ref-crds#helmoptions)
* [Helm.ValuesFrom](/ref-crds#helmoptions)
* [Helm.Values](/ref-crds#helmoptions)
* [Helm.Version](/ref-crds#helmoptions)

  :::warning 重要な情報
  ターゲットカスタマイズを通じてHelmチャートのバージョンを上書きすると、すべてのクラスターに対応するためにチャートのデフォルトバージョンとカスタムバージョンの両方を含むバンドルが作成されます。これにより、Fleetはより大きなバンドルをデプロイすることになります。

  Fleetはバンドルをetcdを介して保存するため、結果としてバンドルサイズがetcdの設定された最大ブロブサイズを超える可能性があるクラスターで問題が発生する可能性があります。詳細については[この問題](https://github.com/rancher/fleet/issues/1650)を参照してください。
  :::

* [Helm.WaitForJobs](/ref-crds#helmoptions)
* [Kustomize.Dir](/ref-crds#kustomizeoptions)
* [YAML.Overlays](/ref-crds#yamloptions)
* [Diff.ComparePatches](/ref-crds#diffoptions)


## 追加の例

生のKubernetes YAML、Helmチャート、Kustomize、およびそれらの組み合わせを使用した例は、[Fleet Examplesリポジトリ](https://github.com/rancher/fleet-examples/)にあります。