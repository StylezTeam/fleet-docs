# クラスター登録の内部

## クラスター登録はどのように機能するのか？

このテキストは、クラスター登録の技術的な詳細を説明します。エージェントによる登録は一般的に使用されないため、ここでは無視します。
[エージェントによる登録](./cluster-registration.md#agent-initiated)は["`ClusterRegistrationToken` ファースト"](./cluster-registration.md#create-cluster-registration-tokens)であり、クラスターを事前に作成することはオプションです。

クラスターの登録方法については、"[下流クラスターの登録](./cluster-registration.md)"を参照してください。

### クラスター ファースト

`fleet-controller`が起動し、ローカルクラスターリソースを「ブートストラップ」することがあります。Rancherでは、ローカルクラスターリソースの作成はfleetclusterコントローラーによって処理されますが、プロセスは同じです。

このプロセスは、ローカルクラスターでも下流クラスターでも同じです。まず、kubeconfigシークレットを参照するクラスターリソースを作成することから始まります。

### 下流クラスターのブートストラップシークレットの作成

このステップでは、`ClusterRegistrationToken`と「インポート」サービスアカウントが`Cluster`リソースに基づいて作成されます。

Fleetコントローラーは[`ClusterRegistrationToken`](https://fleet.rancher.io/architecture#security)を作成し、それが完了するのを待ちます。`ClusterRegistrationToken`は「インポート」サービスアカウントの作成をトリガーし、このアカウントは`ClusterRegistrations`を作成し、システム登録ネームスペース（例：「cattle-fleet-clusters-system」）内の任意のシークレットを読み取ることができます。`import.go`コントローラーは「インポート」サービスアカウントが存在するまで自分自身をキューに入れます。なぜなら、そのアカウントが`fleet-agent-bootstrap`シークレットを作成するために必要だからです。

### Fleetエージェントデプロイメントの作成

Fleetコントローラーは、下流クラスターにFleetエージェントデプロイメントとブートストラップシークレットを作成します。

ブートストラップシークレットには、上流クラスターのAPIサーバーURLが含まれており、上流クラスターにアクセスするためのkubeconfigを構築するために使用されます。これらの値はFleetコントローラーのconfigmapから取得されます。このconfigmapはhelmチャートの一部です。

### Fleetエージェントが登録を開始し、リクエストアカウントにアップグレード

エージェントは「インポート」アカウントを使用してリクエストアカウントにアップグレードします。

Fleetエージェントはすぐに`fleet-agent-bootstrap`シークレットをチェックします。ブートストラップシークレットが存在し、「インポート」kubeconfigが含まれている場合、エージェントは登録を開始します。

その後、エージェントは管理クラスターのfleet-defaultにランダムな番号を持つ最終的な`ClusterRegistration`リソースを作成します。このランダムな番号は登録シークレットの名前に使用されます。

Fleetコントローラーはトリガーを引き、エージェントのサービスアカウントを作成し、クライアントの新しいkubeconfigを持つ「c-*」登録シークレットを作成するための`ClusterRegistration`リクエストを承認しようとします。登録シークレットの名前は`hash("clientID-clientRandom")`です。

新しいkubeconfigは「リクエスト」アカウントを使用します。「リクエスト」アカウントはクラスターのステータス、`BundleDeployments`、および`Contents`にアクセスできます。

### Fleetエージェントが登録され、`BundleDeployments`を監視

この時点でエージェントは完全に登録され、「リクエスト」アカウントを`fleet-agent`シークレットに永続化します。
APIサーバーURLとCAはブートストラップシークレットからコピーされ、これらの値はFleetコントローラーのhelmチャートの値から継承されます。

ブートストラップシークレットは削除されます。エージェントが再起動すると、ブートストラップシークレットがないため再登録は行われません。

エージェントは自分の"[クラスターネームスペース](https://fleet.rancher.io/namespaces#cluster-namespaces)"を監視し、`BundleDeployments`を監視します。この時点でエージェントはワークロードをデプロイする準備が整います。

### 注意点

* 登録は「インポート」アカウントから始まり、「リクエスト」アカウントに移行します。
* fleet-defaultネームスペースにはすべてのクラスター登録があり、「インポート」アカウントは別のネームスペースを使用します。
* エージェントが登録されると、`fleet-controller`はクラスターまたはネームスペースの変更にトリガーを引きます。その後、`manageagent`コントローラーが既存のエージェントデプロイメントを採用するためのバンドルを作成します。エージェントはバンドルに更新され、「generation」環境変数が変更されるため、再起動します。
* ブートストラップシークレットが存在しない場合、エージェントは再登録しません。

## 図

### 登録プロセスとコントローラー

クラスターの登録プロセスの詳細な分析です。これは、新しい下流クラスターまたはローカルクラスターの登録中にコントローラー、リソース、およびサービスアカウントの相互作用を示しています。

これを開始する方法はいくつかあることに注意してください：

* ブートストラップコンフィグの作成。Fleetはローカルエージェントのためにこれを行います。
* kubeconfigを持つ`Cluster`リソースの作成。Rancherは下流クラスターのためにこれを行います。[マネージャーによる登録](./cluster-registration.md#manager-initiated)を参照してください。
* `ClusterRegistrationToken`リソースを作成し、オプションで事前定義された（`clientID`）クラスターのために`Cluster`リソースを作成します。[エージェントによる登録](./cluster-registration.md#agent-initiated)を参照してください。

![Registration](/img/FleetRegistration.svg)

### エージェントデプロイメント中のシークレット

この図は、登録中に作成されるリソースを示し、k8s APIサーバーの構成に焦点を当てています。

`import.go`コントローラーはクラスターの作成/更新イベントにトリガーを引き、エージェントをデプロイします。

**この画像は、APIサーバーURLとCAが登録中にシークレットを通じてどのように伝播するかを示しています：**

図の矢印は、APIサーバーの値がHelmの値から上流クラスターのクラスター登録シークレットにコピーされ、最終的にエージェントのブートストラップシークレットに下流にコピーされる方法を示しています。

特別なケースが1つあります。エージェントがローカル/「ブートストラップ」クラスター用の場合、サーバーの値はクラスターリソースによって参照されるkubeconfigシークレットにも存在します。この場合、kubeconfigシークレットには上流サーバーのURLとCAが含まれており、下流のkubeconfigの隣にあります。設定がkubeconfigシークレットに存在する場合、それらは設定された値を上書きします。

![Registration Secrets](/img/FleetRegistrationSecrets.svg)

## RancherにおけるFleetクラスター登録

Rancherはfleet helmチャートをインストールします。APIサーバーURLとCAは[Rancherの設定から派生](https://github.com/rancher/rancher/blob/release/v2.9/pkg/controllers/dashboard/fleetcharts/controller.go#L111-L112)されます。

Fleetはこれらの値をFleetエージェントに渡し、Fleetコントローラーに接続できるようにします。

### クラスターをRancherにインポート

ユーザーが`curl | kubectl apply`を実行すると、適用されたマニフェストにはrancherエージェントデプロイメントが含まれます。

デプロイメントにはAPI URLとトークンを含むシークレット`cattle-credentials-`が含まれています。

Rancherエージェントが起動し、下流のkubeconfigを上流に報告します。

その後、Rancherはfleetクラスターリソースを作成し、[kubeconfigシークレット](https://github.com/rancher/rancher/blob/871b6d9137246bd93733f01184ea435f40c5d56c/pkg/provisioningv2/kubeconfig/manager.go#L69)を参照します。

👉Fleetはこのkubeconfigを使用して下流クラスターにエージェントをデプロイします。