# コンテナイメージ参照を更新するためのイメージスキャンの使用

フリートのイメージスキャンを使用すると、イメージリポジトリをスキャンし、目的のイメージを取得してGitリポジトリを更新することができます。これにより、マニフェストを手動で更新する必要がなくなります。

:::caution

この機能は実験的な機能と見なされています。

:::

`fleet.yaml`に以下のセクションを追加します。

```yaml
imageScans:
# イメージを取得するポリシーを指定します。semverまたはアルファベット順が使用できます。
- policy:
    # rangeが指定されている場合、範囲内のsemver順で最新のイメージを取得します。
    # semverの使用方法の詳細については、https://github.com/Masterminds/semver を参照してください。
    semver:
      range: "*"
    # 昇順または降順を使用できます。
    alphabetical:
      order: asc

  # スキャンするイメージを指定します。
  image: "your.registry.com/repo/image"

  # タグ名を指定します。同じバンドル内で一意である必要があります。
  tagName: test-scan

  # プライベートレジストリの場合、イメージをプルするためのシークレットを指定します。
  secretRef:
    name: dockerhub-secret

  # スキャン間隔を指定します。
  interval: 5m
```

:::info

fleet.yamlに複数のイメージスキャンを作成することができます。

:::

:::note

Semverはプレリリースバージョン（例: 0.0.1-10）を無視しますが、範囲定義でプレリリースバージョンが明示的に使用されている場合は考慮されます。
例えば、"*" の範囲はプレリリースを無視しますが、">= 0.0.1-10" はそれらを考慮します。

:::

マニフェストファイルに移動し、置き換えたいフィールドを更新します。例えば：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-slave
spec:
  selector:
    matchLabels:
      app: redis
      role: slave
      tier: backend
  replicas: 2
  template:
    metadata:
      labels:
        app: redis
        role: slave
        tier: backend
    spec:
      containers:
      - name: slave
        image: <image>:<tag> # {"$imagescan": "test-scan"}
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
        ports:
        - containerPort: 6379
```

:::note

参照できるtagNameの形式は複数あります。例えば：

`{"$imagescan": "test-scan"}`: フルイメージ名を使用 (foo/bar:tag)

`{"$imagescan": "test-scan:name"}`: タグなしのイメージ名のみを使用 (foo/bar)

`{"$imagescan": "test-scan:tag"}`: イメージタグのみを使用

`{"$imagescan": "test-scan:digest"}`: ダイジェスト付きのフルイメージ名を使用 (foo/bar:tag@sha256...)

:::

fleet.yamlを含むGitRepoを作成します。

```yaml
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: my-repo
  namespace: fleet-local
spec:
  # これを自分のリポジトリに変更します。
  repo: https://github.com/rancher/fleet-examples
  # すべてのイメージを同期し、変更を適用するまでの時間を定義します。
  imageScanInterval: 5m
  # ユーザーはGitリポジトリに書き込みアクセス権を持つシークレットを適切に提供する必要があります。
  clientSecretName: secret
  # コミットパターンを指定します。
  imageScanCommit:
    authorName: foo
    authorEmail: foo@bar.com
    messageTemplate: "update image"
```

新しいイメージタグ、例えば `<image>:<new-tag>` をプッシュしてみてください。しばらく待つと、Gitリポジトリに新しいコミットがプッシュされ、deployment.yamlのタグが変更されるはずです。Gitリポジトリに変更が加えられると、フリートはその変更を読み取り、クラスターに変更をデプロイします。