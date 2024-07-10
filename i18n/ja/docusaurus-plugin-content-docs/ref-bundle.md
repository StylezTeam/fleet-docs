# バンドルリソース

バンドルは、`GitRepo`が作成されたときにFleetによって自動的に作成されます。

リソースの内容は[BundleSpec](./ref-crds#bundlespec)に対応しています。
バンドルリソースの使用方法については[バンドルリソースを作成する](./bundle-add.md)を参照してください。

```yaml
kind: Bundle
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  # 任意の名前を使用できます
  name: my-bundle
  # 単一のクラスターの場合はfleet-localを使用し、それ以外の場合は
  # 任意のネームスペースを使用します
  namespace: fleet-local
spec:
  # ネームスペースを指定しないリソースに使用されるネームスペース。
  # このフィールドは特定のネームスペースへのデプロイを強制またはロックダウンするために使用されません。
  # defaultNamespace: test

  # 存在する場合、すべてのリソースをこのネームスペースに割り当て、
  # クラスター範囲のリソースが存在する場合、デプロイは失敗します。
  # targetNamespace: app

  # デプロイのためのKustomizeオプション。kustomization.yamlファイルを含むディレクトリなど。
  # kustomize: ...

  # デプロイのためのHelmオプション。チャート名、リポジトリ、値など。
  # helm: ...

  # このデプロイを実行するために使用されるServiceAccount。
  # serviceAccount: sa

  # ForceSyncGenerationは再デプロイを強制するために使用されます。
  # forceSyncGeneration: 0

  # YAMLオプション。生のYAMLを使用する場合、リソースを置き換えまたはパッチするために使用されるoverlays/{name}にマップされる名前。
  # yaml: ...

  # Diffは、実行時に修正されたオブジェクトの変更状態を無視するために使用できます。
  # 特定のコミットまたはタグも監視できます。
  #
  # diff: ...

  # バンドルを削除する際にデプロイされたリソースを保持するために使用できます。
  # keepResources: false

  # trueに設定すると、BundleDeploymentsの更新が停止します。同期が取れていないとマークされます。
  # paused: false

  # パーティション、カナリア、クラスターの可用性のパーセンテージを定義することで、バンドルのロールアウトを制御します。
  # rolloutStrategy: ...

  # デプロイされるgitリポジトリからの実際のリソースを含みます。
  resources:
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

  # Fleetをマルチクラスター形式で実行する場合にデプロイするターゲットクラスター。
  # 詳細については、「下流クラスターへのマッピング」ドキュメントを参照してください。
  #
  # targets: ...

  # このフィールドはFleetによって内部的に使用され、手動で変更しないでください。
  # Fleetは、GitRepoのバンドルが作成されるときにすべてのターゲットをtargetRestrictionsにコピーします。
  # targetRestrictions: ...

  # このバンドルがデプロイされる前に準備が必要なバンドルを参照します。
  # dependsOn: ...

```