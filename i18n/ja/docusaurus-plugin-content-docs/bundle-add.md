# バンドルリソースを作成する

`GitRepo`が作成されると、バンドルはFleetによって自動的に作成されます。ほとんどの場合、ユーザーが手動で`バンドル`を作成する必要はありません。Gitリポジトリからリソースをデプロイしたい場合は、代わりに[GitRepo](https://fleet.rancher.io/gitrepo-add)を使用してください。

Gitリポジトリを使用せずにリソースをデプロイしたい場合は、このガイドに従って`バンドル`を作成してください。

`GitRepo`を作成すると、FleetはGitリポジトリからリソースを取得し、それらをバンドルに追加します。`バンドル`を作成する場合、リソースは`バンドル`のSpecに明示的に指定する必要があります。リソースはgzで圧縮することができます。RancherがGoコードで圧縮を使用する例については[こちら](https://github.com/rancher/rancher/blob/v2.7.3/pkg/controllers/provisioningv2/managedchart/managedchart.go#L149-L153)を参照してください。

ダウンストリームクラスターにデプロイしたい場合は、ターゲットを定義する必要があります。ターゲットは`GitRepo`のターゲットと同様に機能します。[ダウンストリームクラスターへのマッピング](https://fleet.rancher.io/gitrepo-targets#defining-targets)を参照してください。

次の例では、ローカルクラスターにnginxの`Deployment`を作成します：

```yaml
kind: Bundle
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  # 任意の名前を使用できます
  name: my-bundle
  # 単一クラスターの場合はfleet-localを使用し、それ以外の場合は
  # 任意の名前空間を使用します
  namespace: fleet-local
spec:
  resources:
  # デプロイされるすべてのリソースのリスト
  - content: |
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: nginx-deployment
        labels:
          app: nginx
      spec:
        replicas: 3
        selector:
          matchLabels:
            app: nginx
        template:
          metadata:
            labels:
              app: nginx
          spec:
            containers:
              - name: nginx
                image: nginx:1.14.2
                ports:
                  - containerPort: 80
    name: nginx.yaml
  targets:
  - clusterName: local

```

## 制限事項

Helmチャートのダウンロードに関連するHelmオプションは無視されます。Helmチャートはバンドルを作成するfleet-cliによってダウンロードされます。したがって、バンドルにはチャートからのすべてのリソースが含まれている必要があります。そのため、バンドルは以下を無視します：

* `spec.helm.repo`
* `spec.helm.charts`

リソース内で`fleet.yaml`を使用することはできません。これはバンドルを作成するためにfleet-cliによってのみ使用されます。

`spec.targetRestrictions`フィールドは役に立ちません。これは`spec.targets`で指定されたターゲットの許可リストであり、バンドル内でターゲットが明示的に指定されているため必要ありません。空の`targetRestrictions`はデフォルトで許可されます。

## Helmチャートをバンドルに変換する

Fleet CLIを使用してHelmチャートをバンドルに変換できます。

例えば、「external secrets」オペレーターのチャートを次のようにダウンロードして変換できます：
```
cat > targets.yaml <<EOF
targets:
- clusterSelector: {}
EOF

mkdir app
cat > app/fleet.yaml <<EOF
defaultNamespace: external-secrets
helm:
  repo: https://charts.external-secrets.io
  chart: external-secrets
EOF

fleet apply --compress --targets-file=targets.yaml -n fleet-default -o - external-secrets app > eso-bundle.yaml

kubectl apply -f eso-bundle.yaml
```

`targets.yaml`でデプロイしたいすべてのクラスターに一致するクラスターセレクターを使用してください。

[Fleet: Multi-Cluster Deployment with the Help of External Secrets](https://www.suse.com/c/rancher_blog/fleet-multi-cluster-deployment-with-the-help-of-external-secrets/)のブログ記事には、さらに詳しい情報があります。