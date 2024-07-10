# マルチユーザー設定

Fleetは可能な限りKubernetes RBACを使用します。

RBACに加えて、[`GitRepoRestriction`](./namespaces.md#restricting-gitrepos)リソースがあり、これは名前空間内のGitRepoリソースを制御するために使用できます。

マルチユーザーのFleet設定は次のようになります：

* テナントは名前空間を共有せず、各テナントは上流クラスターに1つ以上の名前空間を持ち、そこでGitRepoリソースを作成できます。
* テナントはクラスター全体のリソースをデプロイできず、下流クラスターの特定の名前空間に制限されます。
* クラスターは別の名前空間にあります。

![共有クラスター](/img/FleetSharedClusters.svg)

:::warning 重要な情報

テナントの隔離は完全ではなく、Kubernetes RBACが正しく設定されていることに依存します。オペレーターによる手動設定がない場合、テナントはクラスター全体のリソースをデプロイすることができます。利用可能なFleetの制限があっても、ユーザーは名前空間に制限されますが、名前空間自体は多くの隔離を提供しません。例えば、彼らは好きなだけリソースを消費することができます。

しかし、既存のFleetの制限により、ユーザーはクラスターを共有し、リソースを競合なくデプロイすることができます。

:::

## 例：スタンドアロンのFleet

これは、'fleetuser'というユーザーを作成し、'project1'名前空間内のGitRepoリソースのみを管理できるようにします。

    kubectl create serviceaccount fleetuser
    kubectl create namespace project1
    kubectl create -n project1 role fleetuser --verb=get --verb=list --verb=create --verb=delete --resource=gitrepos.fleet.cattle.io
    kubectl create -n project1 rolebinding fleetuser --serviceaccount=default:fleetuser --role=fleetuser

複数の名前空間にアクセスを許可したい場合、2つのロールバインディングを持つ単一のクラスター役割を使用できます：

    kubectl create clusterrole fleetuser --verb=get --verb=list --verb=create --verb=delete --resource=gitrepos.fleet.cattle.io
    kubectl create -n project1 rolebinding fleetuser --serviceaccount=default:fleetuser --clusterrole=fleetuser
    kubectl create -n project2 rolebinding fleetuser --serviceaccount=default:fleetuser --clusterrole=fleetuser

これにより、テナントは他のテナントのGitRepoリソースに干渉することができなくなります。なぜなら、彼らは他の名前空間にアクセスできないからです。

## 例：RancherでのFleet

新しいFleetワークスペースが作成されると、Rancherローカルクラスター内に同じ名前の対応する名前空間が自動的に生成されます。
特定のワークスペースでFleetリソースを表示およびデプロイするには、ユーザーに少なくとも次の権限が必要です：
- ローカルクラスター内の`fleetworkspace`クラスター全体のリソースをリスト/取得する権限
- ローカルクラスター内のワークスペースのバックアップ名前空間でFleetリソース（`bundles`、`gitrepos`など）を作成する権限

`project1`および`project2`のFleetワークスペースにFleetリソースをデプロイする権限を付与しましょう：

- `project1`および`project2`のFleetワークスペースを作成するには、[Rancher UI](https://ranchermanager.docs.rancher.com/integrations-in-rancher/fleet/overview#accessing-fleet-in-the-rancher-ui)で行うか、次のYAMLリソースを使用します：

```
apiVersion: management.cattle.io/v3
kind: FleetWorkspace
metadata:
  name: project1
```

```
apiVersion: management.cattle.io/v3
kind: FleetWorkspace
metadata:
  name: project2
```

- `project1`および`project2`のFleetワークスペースにFleetリソースをデプロイする権限を付与する`GlobalRole`を作成します：

```
apiVersion: management.cattle.io/v3
kind: GlobalRole
metadata:
  name: fleet-projects1and2
namespacedRules:
  project1:
    - apiGroups:
        - fleet.cattle.io
      resources:
        - gitrepos
        - bundles
        - clusterregistrationtokens
        - gitreporestrictions
        - clusters
        - clustergroups
      verbs:
        - '*'
  project2:
    - apiGroups:
        - fleet.cattle.io
      resources:
        - gitrepos
        - bundles
        - clusterregistrationtokens
        - gitreporestrictions
        - clusters
        - clustergroups
      verbs:
        - '*'
rules:
  - apiGroups:
      - management.cattle.io
    resourceNames:
      - project1
      - project2
    resources:
      - fleetworkspaces
    verbs:
      - '*'
```

`GlobalRole`をユーザーまたはグループに割り当てます。詳細は[Rancherドキュメント](https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/authentication-permissions-and-global-configuration/manage-role-based-access-control-rbac/global-permissions#configuring-global-permissions-for-individual-users)で確認できます。

これで、ユーザーはRancherの`Continuous Delivery`タブにアクセスし、`project1`および`project2`のワークスペースにリソースをデプロイできるようになります。

## クラスターへのアクセスを許可する

これは、'fleetuser'によって作成されたすべてのGitRepoが`team: one`ラベルを持っていることを前提としています。異なるラベルを使用して、異なるクラスター名前空間を選択することもできます。

各ユーザーの名前空間内で、管理者として[`BundleNamespaceMapping`](./namespaces.md#cross-namespace-deployments)を作成します。

    kind: BundleNamespaceMapping
    apiVersion: fleet.cattle.io/v1alpha1
    metadata:
      name: mapping
      namespace: project1

    # ラベルで一致するバンドル
    # ラベルはfleet.yamlの# labelsフィールドまたは
    # GitRepoのmetadata.labelsフィールドで定義されます
    bundleSelector:
      matchLabels:
        team: one
        # または1つのリポジトリをターゲットにする
        #fleet.cattle.io/repo-name: simpleapp

    # ラベルで一致するクラスターを含む名前空間
    namespaceSelector:
      matchLabels:
        kubernetes.io/metadata.name: fleet-default
        # ラベルは名前空間にあります
        #workspace: prod

GitRepoリソースの[`target`セクション](./gitrepo-targets.md)を使用して、一部の一致するクラスターにのみデプロイすることができます。

## 下流クラスターへのアクセス制限

管理者は各名前空間に`GitRepoRestriction`を作成することで、テナントをさらに制限できます。

    kind: GitRepoRestriction
    apiVersion: fleet.cattle.io/v1alpha1
    metadata:
      name: restriction
      namespace: project1

    allowedTargetNamespaces:
      - project1simpleapp

これにより、他のテナントに干渉する可能性のあるクラスター全体のリソースの作成が拒否され、'project1simpleapp'名前空間へのデプロイに制限されます。

## 例：GitRepoリソース

管理者アクセスなしでテナントによって作成されたGitRepoリソースは次のようになります：

    kind: GitRepo
    apiVersion: fleet.cattle.io/v1alpha1
    metadata:
      name: simpleapp
      namespace: project1
      labels:
        team: one

    spec:
      repo: https://github.com/rancher/fleet-examples
      paths:
      - bundle-diffs

      targetNamespace: project1simpleapp

      # 上流/ローカルクラスターと一致しない、機能しない
      targets:
      - name: dev
        clusterSelector:
          matchLabels:
            env: dev

これには`team: one`ラベルと必要な`targetNamespace`が含まれています。

前述の`BundleNamespaceMapping`と組み合わせることで、'fleet-default'名前空間内の`env: dev`ラベルを持つすべてのクラスターをターゲットにします。

:::note

`BundleNamespaceMappings`はローカルクラスターでは機能しないため、ターゲットにしないようにしてください。

:::
