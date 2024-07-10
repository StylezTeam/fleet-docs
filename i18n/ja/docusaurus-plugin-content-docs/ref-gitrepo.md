# GitRepo リソース

GitRepo リソースは、Git リポジトリの説明、アクセス方法、およびバンドルの場所を示します。

リソースの内容は [GitRepoSpec](./ref-crds#gitrepospec) に対応しています。
GitRepo リソースの使用方法、例えばプライベートリポジトリの監視方法については、[GitRepo リソースの作成](./gitrepo-add.md) を参照してください。

```yaml
kind: GitRepo
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  # 任意の名前を使用できます
  name: my-repo
  # 単一クラスターの場合は fleet-local を使用し、それ以外の場合は任意のネームスペースを使用します
  namespace: fleet-local
spec:
  # これは HTTPS または git URL である可能性があります。git URL を使用する場合は、
  # 認証情報を提供するために clientSecretName を設定する必要があるかもしれません。
  # repo は監視されるリポジトリに対して唯一必要なパラメータです。
  #
  repo: https://github.com/rancher/fleet-examples

  # すべてのリソースがこのターゲットネームスペースに移動することを強制します。
  # クラスター範囲のリソースが見つかった場合、デプロイメントは失敗します。
  #
  # targetNamespace: app1

  # 任意のブランチを監視できます。このフィールドはオプションです。指定しない場合、
  # ブランチは master と見なされます。
  #
  # branch: master

  # 特定のコミットまたはタグも監視できます。
  #
  # revision: v0.3.0

  # プライベート Git リポジトリの場合、clientSecretName を提供する必要があります。
  # デフォルトのシークレットは GitRepoRestriction タイプを使用してネームスペースレベルで設定できます。
  # シークレットは "kubernetes.io/ssh-auth" または "kubernetes.io/basic-auth" タイプである必要があります。
  # シークレットは GitRepo と同じネームスペースにあると見なされます。
  #
  # clientSecretName: my-ssh-key

  # fleet.yaml に認証が必要なプライベート Helm リポジトリが含まれている場合、
  # K8s シークレットで認証情報を提供し、ここに指定します。
  # 注意: 認証情報はこの gitrepo から参照されるすべてのリポジトリに送信されます。
  # 詳細は以下のセクションを参照してください。
  #
  # helmSecretName: my-helm-secret

  # helmSecretName からの Helm 認証情報は、Helm リポジトリ URL がこの正規表現に一致する場合に使用されます。
  # 空または指定されていない場合、認証情報は常に使用されます。
  #
  # helmRepoURLRegex: https://charts.rancher.io/*

  # 各パスのプライベート Helm リポジトリの認証シークレットを含みます。
  # 詳細は [GitRepo リソースの作成](.gitrepo-add#use-different-helm-credentials-for-each-path) を参照してください。
  #
  # helmSecretNameForPaths: multi-helm-secret

  # 自己署名証明書の追加の ca-bundle を追加するには、caBundle に base64 エンコードされた pem データを入力できます。
  # 例: `cat /path/to/ca.pem | base64 -w 0`
  #
  # caBundle: my-ca-bundle

  # Git リポジトリの SSL 検証を無効にします
  #
  # insecureSkipTLSVerify: true

  # Git リポジトリ内の複数のパスを一度に読み取ることができます。
  # 以下のフィールドはパスの配列であることが期待され、パスのグロビングをサポートします（例: some/*/path）
  #
  # 例:
  # paths:
  # - single-path
  # - multiple-paths/*
  paths:
  - simple

  # PollingInterval は、Fleet が Git リポジトリをチェックする頻度を設定します。デフォルトは 15 秒です。
  # これをゼロに設定してもポーリングは無効になりません。結果として 15 秒の間隔になります。
  # Git リポジトリのチェックには CPU コストがかかるため、複数の Git リポジトリを使用する場合やそれ以上の場合、
  # この値を上げることで fleetcontroller の CPU 使用率を下げるのに役立ちます。
  #
  # pollingInterval: 15s

  # disablePolling が true に設定されている場合、Git リポジトリは定期的にチェックされません。
  # 代わりに Webhook のみを使用します。
  # 詳細は [ポーリングの代わりに Webhook を使用する](https://fleet.rancher.io/webhook) を参照してください。
  # disablePolling: false

  # Paused は、Git の変更がクラスターに伝播されず、代わりにリソースが OutOfSync としてマークされるようにします。
  #
  # paused: false

  # Git からのコンテンツの再デプロイを強制するには、この番号を増やします。
  #
  # forceSyncGeneration: 0

  # このデプロイメントを実行するために使用されるサービスアカウント。
  # これは、cattle-fleet-system ネームスペース内のダウンストリームクラスターに存在するサービスアカウントの名前です。
  # このサービスアカウントは既に存在することが前提となっているため、事前に作成しておく必要があります。
  # おそらく、Fleet マネージャーに登録された別の Git リポジトリから来るでしょう。
  #
  # serviceAccount: moreSecureAccountThanClusterAdmin

  # Fleet をマルチクラスター方式で実行する場合にデプロイするターゲットクラスター。
  # 詳細は「ダウンストリームクラスターへのマッピング」ドキュメントを参照してください。
  # 空の場合、"default" クラスターグループが使用されます。
  #
  # targets: ...

  # ドリフト修正は、Fleet によって管理されるリソースに対して行われた外部の変更を削除します。
  # これはデフォルトで 3 方向マージ戦略を使用する Helm ロールバックを実行します。
  # 強制が有効になっている場合、PUT リクエストを行うことで全リソースを更新しようとします。
  # 3 方向戦略的マージは、配列内のアイテムを更新する際に失敗する可能性があります。
  # これは新しいアイテムを追加しようとするためです。これを修正するには強制を使用します。
  # 強制が有効になっている場合、リソースが再作成される可能性があることに注意してください。
  # 失敗したロールバックは、keepFailHistory が true に設定されていない限り、Helm 履歴から削除されます。
  #
  #  correctDrift:
  #    enabled: false
  #    force: false #警告: true に設定するとリソースが再作成される可能性があります
  #    keepFailHistory: false
```