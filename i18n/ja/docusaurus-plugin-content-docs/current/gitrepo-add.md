# GitRepo リソースの作成

## GitRepo インスタンスの作成

Git リポジトリは Kubernetes で `GitRepo` リソースを作成することで登録されます。例については[デプロイメント作成チュートリアル](./tut-deployment.md)を参照してください。

[Git リポジトリの内容](./gitrepo-content.md)には、Git リポジトリの内容に関する詳細が記載されています。

GitRepo カスタムリソースの利用可能なフィールドは、[GitRepo リソースリファレンス](./ref-gitrepo.md)に記載されています。

### 適切なネームスペース

Git リポジトリは `GitRepo` カスタムリソースタイプを使用して Fleet マネージャーに追加されます。`GitRepo` タイプはネームスペースに属します。デフォルトでは、Rancher は 2 つの Fleet ワークスペースを作成します: **fleet-default** と **fleet-local**。

- `fleet-default` には、すでに Rancher を通じて登録されているすべてのダウンストリームクラスターが含まれます。
- `fleet-local` には、デフォルトでローカルクラスターが含まれます。

Fleet を[シングルクラスター](./concepts.md)スタイルで使用している場合、ネームスペースは常に **fleet-local** になります。`fleet-local` ネームスペースの詳細については[こちら](https://fleet.rancher.io/namespaces/#fleet-local)を参照してください。

[マルチクラスター](./concepts.md)スタイルの場合、正しいターゲットクラスターにマッピングされる正しいリポジトリを使用するようにしてください。

## ワークロードのネームスペースを上書きする

`targetNamespace` フィールドはバンドル内の任意のネームスペースを上書きします。デプロイメントにクラスター範囲のリソースが含まれている場合、失敗します。

このフィールドは他のすべてのネームスペース定義よりも優先されます：

`gitRepo.targetNamespace > fleet.yaml namespace > workload のマニフェスト内の namespace > fleet.yaml defaultNamespace`

ワークロードのネームスペース定義は、`GitRepoRestriction` リソース内の `allowedTargetNamespaces` で制限できます。

## プライベート Git リポジトリの追加

Fleet はプライベートリポジトリに対して http および ssh 認証キーの両方をサポートしています。これを使用するには、同じネームスペースにシークレットを作成する必要があります。

例えば、プライベート ssh キーを生成するには

```text
ssh-keygen -t rsa -b 4096 -m pem -C "user@email.com"
```

注意: プライベートキーの形式は `EC PRIVATE KEY`、`RSA PRIVATE KEY` または `PRIVATE KEY` であり、パスフレーズを含まない必要があります。

プライベートキーをシークレットに入れ、GitRepo が存在するネームスペースを使用します：

```text
kubectl create secret generic ssh-key -n fleet-default --from-file=ssh-privatekey=/file/to/private/key  --type=kubernetes.io/ssh-auth
```

次に、リポジトリ定義で `clientSecretName` を指定する必要があります：

```text
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: sample-ssh
  # このネームスペースは特別で、ローカルクラスターにデプロイするように自動的に設定されます
  namespace: fleet-local
spec:
  # このリポジトリからのすべてのものがこのクラスターで実行されます。信じてくれますよね？
  repo: "git@github.com:rancher/fleet-examples"
  # または
  # repo: "ssh://git@github.com/rancher/fleet-examples"
  clientSecretName: ssh-key
  paths:
  - simple
```

:::caution

パスフレーズ付きのプライベートキーはサポートされていません。

:::

:::caution

キーは PEM 形式である必要があります。

:::

### Known hosts

:::warning

シークレットに1つ以上の公開鍵を追加しない場合、任意のサーバーの公開鍵が信頼されて追加されます。 (`ssh -o stricthostkeychecking=accept-new` が使用されます)

:::

Fleet は `known_hosts` を ssh シークレットに追加することをサポートしています。以下はその例です：

公開鍵ハッシュを取得します（例として GitHub を使用）

```text
ssh-keyscan -H github.com
```

そしてシークレットに追加します：

```text
apiVersion: v1
kind: Secret
metadata:
  name: ssh-key
type: kubernetes.io/ssh-auth
stringData:
  ssh-privatekey: <private-key>
  known_hosts: |-
    |1|YJr1VZoi6dM0oE+zkM0do3Z04TQ=|7MclCn1fLROZG+BgR4m1r8TLwWc= ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAq2A7hRGmdnm9tUDbO9IDSwBK6TbQa+PXYPCPy6rbTrTtw7PHkccKrpp0yVhp5HdEIcKr6pLlVDBfOLX9QUsyCOV0wzfjIJNlGEYsdlLJizHhbn2mUjvSAHQqZETYP81eFzLQNnPHt4EVVUh7VfDESU84KezmD5QlWpXLmvU31/yMf+Se8xhHTvKSCZIFImWwoG6mbUoWf9nzpIoaSjB+weqqUUmpaaasXVal72J+UX2B+2RPW3RcT0eOzQgqlJL3RKrTJvdsjE3JEAvGq3lGHSZXy28G3skua2SmVi/w4yCE6gbODqnTWlg7+wC604ydGXA8VJiS5ap43JXiUFFAaQ==
```

### HTTP 認証の使用

ユーザー名とパスワードを含むシークレットを作成します。必要に応じて、パスワードを個人用アクセストークンに置き換えることができます。詳細は[GitHub の HTTP シークレット](./troubleshooting#http-secrets-in-github)を参照してください。

```text
kubectl create secret generic basic-auth-secret -n fleet-default --type=kubernetes.io/basic-auth --from-literal=username=$user --from-literal=password=$pat
```

SSH と同様に、`clientSecretName` を介して GitRepo リソースにシークレットを参照します。

```yaml
spec:
  repo: https://github.com/fleetrepoci/gitjob-private.git
  branch: main
  clientSecretName: basic-auth-secret
```

## プライベート Helm リポジトリの使用

:::warning
資格情報は、gitrepo リソースによって参照されるすべての Helm リポジトリに無条件で使用されます。
資格情報が漏洩しないように、公開リポジトリとプライベートリポジトリを混在させないでください。[各パスに対して異なる Helm 資格情報を使用する](#use-different-helm-credentials-for-each-path)か、別々の gitrepo に分割するか、`helmRepoURLRegex` を使用して資格情報の範囲を特定のサーバーに制限してください。
:::

プライベート Helm リポジトリの場合、ユーザーは以下のキーを持つシークレットを参照できます：

1. Helm HTTP リポジトリが基本認証の背後にある場合、基本 http 認証用の `username` と `password`。

2. Helm リポジトリがカスタム CA を使用している場合、カスタム CA バンドル用の `cacerts`。

3. リポジトリが ssh プロトコルを使用している場合、ssh プライベートキー用の `ssh-privatekey`。現在、パスフレーズ付きのプライベートキーはサポートされていません。

例えば、kubectl でシークレットを追加するには、以下のコマンドを実行します：

```text
kubectl create secret -n $namespace generic helm --from-literal=username=foo --from-literal=password=bar --from-file=cacerts=/path/to/cacerts --from-file=ssh-privatekey=/path/to/privatekey.pem
```

シークレットが作成された後、`gitRepo.spec.helmSecretName` にシークレットを指定します。シークレットは gitrepo と同じネームスペースで作成されていることを確認してください。

### 各パスに対して異なる Helm 資格情報を使用する

:::info
`gitRepo.spec.helmSecretName` は `gitRepo.spec.helmSecretNameForPaths` が提供されている場合は無視されます。
:::

`GitRepo` に定義された各パスの資格情報を含む `secrets-path.yaml` ファイルを作成します。このファイルに存在しないパスには資格情報は使用されません。
パスは git リポジトリ内のバンドル（つまり `fleet.yaml` ファイルを含むフォルダー）への実際のパスであり、`paths:` のエントリよりも多くのセグメントを持つ場合があります。

例：

```yaml
path-one: # リポジトリ内に path-one パスが存在する必要があります
  username: user
  password: pass
path-two:  # リポジトリ内に path-two パスが存在する必要があります
  username: user2
  password: pass2
  caBundle: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCiAgICBNSUlEblRDQ0FvV2dBd0lCQWdJVUNwMHB2SVJTb2c0eHJKN2Q1SUI2ME1ka0k1WXdEUVlKS29aSWh2Y05BUUVMCiAgICBCUUF3WGpFTE1Ba0dBMVVFQmhNQ1FWVXhFekFSQmdOVkJBZ01DbE52YldVdFUzUmhkR1V4SVRBZkJnTlZCQW9NCiAgICBHRWx1ZEdWeWJtVjBJRmRwWkdkcGRITWdVSFI1SUV4MFpERVhNQlVHQTFVRUF3d09jbUZ1WTJobGNpNXRlUzV2CiAgICBjbWN3SGhjTk1qTXdOREkzTVRVd056VXpXaGNOTWpnd05ESTFNVFV3TnpVeldqQmVNUXN3Q1FZRFZRUUdFd0pCCiAgICBWVEVUTUJFR0ExVUVDQXdLVTI5dFpTMVRkR0YwWlRFaE1COEdBMVVFQ2d3WVNXNTBaWEp1WlhRZ1YybGtaMmwwCiAgICBjeUJRZEhrZ1RIUmtNUmN3RlFZRFZRUUREQTV5WVc1amFHVnlMbTE1TG05eVp6Q0NBU0l3RFFZSktvWklodmNOCiAgICBBUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTXBvZE5TMDB6NDc1dnVSc2ZZcTFRYTFHQVl3QU92anV4MERKTHY5CiAgICBrZFhwT091dGdjMU8yWUdqNUlCVGQzVmpISmFJYUg3SDR2Rm84RlBaMG9zcU9YaFg3eUM4STdBS3ZhOEE5VmVmCiAgICBJVXp6Vlo1cCs1elNxRjdtZTlOaUNiL0pVSkZLT0ZsTkF4cjZCcXhoMEIyN1VZTlpjaUIvL1V0L0I2eHJuVE55CiAgICBoRzJiNzk4bjg4bFZqY3EzbEE0djFyM3VzWGYxVG5aS2t2UEN4ZnFHYk5OdTlpTjdFZnZHOWoyekdHcWJvcDRYCiAgICBXY3VSa3N3QkgxZlRNS0ZrbGcrR1VsZkZPMGFzL3phalVOdmdweTlpdVBMZUtqZTVWcDBiMlBLd09qUENpV2d4CiAgICBabDJlVDlNRnJjV0F3NTg3emE5NDBlT1Era2pkdmVvUE5sU2k3eVJMMW96YlRka0NBd0VBQWFOVE1GRXdIUVlECiAgICBWUjBPQkJZRUZEQkNkYjE4M1hsU0tWYzBxNmJSTCt0dVNTV3lNQjhHQTFVZEl3UVlNQmFBRkRCQ2RiMTgzWGxTCiAgICBLVmMwcTZiUkwrdHVTU1d5TUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3RFFZSktvWklodmNOQVFFTEJRQURnZ0VCCiAgICBBQ1BCVERkZ0dCVDVDRVoxd1pnQmhKdm9GZTk2MUJqVCtMU2RxSlpsSmNRZnlnS0hyNks5ZmZaY1ZlWlBoMVU0CiAgICB3czBuWGNOZiszZGJlTjl4dVBiY0VqUWlQaFJCcnRzalE1T1JiVHdYWEdBdzlYbDZYTkl6YjN4ZDF6RWFzQXZPCiAgICBJMjM2ZHZXQ1A0dWoycWZqR0FkQjJnaXU2b2xHK01CWHlneUZKMElzRENraldLZysyWEdmU3lyci9KZU1vZlFBCiAgICB1VU9wcFVGdERYd0lrUW1VTGNVVUxWcTdtUVNQb0lzVkNNM2hKNVQzczdUSWtHUDZVcGVSSjgzdU9LbURYMkRHCiAgICBwVWVQVHBuVWVLOVMzUEVKTi9XcmJSSVd3WU1OR29qdDRKWitaK1N6VE1aVkh0SlBzaGpjL1hYOWZNU1ZXQmlzCiAgICBQRW5MU256MDQ4OGFUQm5SUFlnVXFsdz0KICAgIC0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0t
  sshPrivateKey: ICAgIC0tLS0tQkVHSU4gQ0VSVElGSUNBVEUtLS0tLQogICAgTUlJRFF6Q0NBaXNDRkgxTm5YUWI5SlV6anNBR3FSc3RCYncwRlFpak1BMEdDU3FHU0liM0RRRUJDd1VBTUY0eAogICAgQ3pBSkJnTlZCQVlUQWtGVk1STXdFUVlEVlFRSURBcFRiMjFsTFZOMFlYUmxNU0V3SHdZRFZRUUtEQmhKYm5SbAogICAgY201bGRDQlhhV1JuYVhSeklGQjBlU0JNZEdReEZ6QVZCZ05WQkFNTURuSmhibU5vWlhJdWJYa3ViM0puTUI0WAogICAgRFRJek1EUXlOekUxTVRBMU5Gb1hEVEkwTURReU5qRTFNVEExTkZvd1hqRUxNQWtHQTFVRUJoTUNRVlV4RXpBUgogICAgQmdOVkJBZ01DbE52YldVdFUzUmhkR1V4SVRBZkJnTlZCQW9NR0VsdWRHVnlibVYwSUZkcFpHZHBkSE1nVUhSNQogICAgSUV4MFpERVhNQlVHQTFVRUF3d09jbUZ1WTJobGNpNXRlUzV2Y21jd2dnRWlNQTBHQ1NxR1NJYjNEUUVCQVFVQQogICAgQTRJQkR3QXdnZ0VLQW9JQkFRRGd6UUJJTW8xQVFHNnFtYmozbFlYUTFnZjhYcURTbjdyM2lGcVZZZldDVWZOSwogICAgaGZwampTRGpOMmRWWEV2UXA3R0t3akFHUElFbXR5RmxyUW5rUGtnTGFSaU9jSDdNN0p2c3ZIa0
前の例では、ユーザー名 `user` の資格情報が `path-one` のパスに使用され、ユーザー名 `user2` の資格情報が `path-two` のパスに使用されます。

`caBundle` と `sshPrivateKey` は base64 エンコードされている必要があります。

:::note
もし["rancher-backups"](https://ranchermanager.docs.rancher.com/how-to-guides/new-user-guides/backup-restore-and-disaster-recovery/back-up-rancher)を使用していて、このシークレットをバックアップに含めたい場合は、シークレットにラベル `resources.cattle.io/backup: true` を追加してください。その場合、機密資格情報を保護するためにバックアップを暗号化することを忘れないでください。
:::

# トラブルシューティング

Fleetのトラブルシューティングセクションは[こちら](./troubleshooting.md)を参照してください。