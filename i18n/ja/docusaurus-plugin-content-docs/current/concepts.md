# コアコンセプト

Fleetは基本的に、単一のKubernetesクラスターまたは大規模なKubernetesクラスターのデプロイメントを管理するためのGitOpsを管理するためのKubernetesカスタムリソース定義（CRD）とコントローラーのセットです。

:::info

CRDの命名規則について詳しくは、[こちら](./troubleshooting.md#naming-conventions-for-crds)をクリックしてください。

:::

以下は、このドキュメント全体で役立つFleetのいくつかのコンセプトです：

* **Fleet Manager**: GitからKubernetes資産のデプロイメントをオーケストレーションする集中型コンポーネントです。マルチクラスターセットアップでは、通常、専用のKubernetesクラスターになります。単一クラスターセットアップでは、FleetマネージャーはGitOpsで管理している同じクラスター上で実行されます。
* **Fleet controller**: GitOpsをオーケストレーションするFleetマネージャー上で実行されるコントローラーです。実際には、FleetマネージャーとFleetコントローラーはほぼ同義で使用されます。
* **Single Cluster Style**: マネージャーとダウンストリームクラスターが同じクラスターであるFleetのインストールスタイルです。これは、GitOpsを迅速に開始するための非常にシンプルなパターンです。
* **Multi Cluster Style**: 多数のダウンストリームクラスターを管理する中央マネージャーを持つFleetの実行スタイルです。
* **Fleet agent**: 管理されるすべてのダウンストリームクラスターは、Fleetマネージャーと通信するエージェントを実行します。このエージェントは、ダウンストリームクラスターで実行される別のKubernetesコントローラーセットです。
* **GitRepo**: Fleetによって監視されるGitリポジトリは、`GitRepo`タイプで表されます。

>**ダウンストリームクラスターの構成管理にFleetを使用する場合の`GitRepo`カスタムリソースによるインストール順序の例:**
>
> 1. [Calico](https://github.com/projectcalico/calico) CRDとコントローラーをインストールします。
> 2. 1つまたは複数のクラスター全体のグローバルネットワークポリシーを設定します。
> 3. [GateKeeper](https://github.com/open-policy-agent/gatekeeper)をインストールします。**クラスターラベル**と**オーバーレイ**は、バンドルの各部分がどのクラスターに適用されるかを決定するため、Fleetの重要な機能です。
> 4. イングレスとシステムデーモンをセットアップおよび構成します。

* **Bundle**: Gitからのリソースのオーケストレーションに使用される内部ユニットです。`GitRepo`がスキャンされると、1つ以上のバンドルが生成されます。バンドルはクラスターにデプロイされるリソースのコレクションです。`Bundle`はFleetで使用される基本的なデプロイメントユニットです。バンドルの内容はKubernetesマニフェスト、Kustomize構成、またはHelmチャートである可能性があります。ソースに関係なく、内容はエージェントによって動的にHelmチャートにレンダリングされ、Helmリリースとしてダウンストリームクラスターにインストールされます。

    - **バンドルのライフサイクル**を見るには、[こちら](./ref-bundle-stages.md)をクリックしてください。

* **BundleDeployment**: `Bundle`がクラスターにデプロイされると、`Bundle`のインスタンスは`BundleDeployment`と呼ばれます。`BundleDeployment`は、特定のクラスターでのその`Bundle`の状態を表し、そのクラスター固有のカスタマイズを含みます。Fleetエージェントは、エージェントが管理しているクラスターのために作成された`BundleDeployment`リソースのみを認識します。

    - Fleetカスタマイズを使用してクラスター間でKubernetesマニフェストをデプロイする方法の例については、[こちら](./gitrepo-targets.md#customization-per-cluster)をクリックしてください。

* **Downstream Cluster**: Fleetがマニフェストをデプロイするクラスターはダウンストリームクラスターと呼ばれます。単一クラスターの使用例では、FleetマネージャーKubernetesクラスターはマネージャーとダウンストリームクラスターの両方を同時に兼ねます。
* **Cluster Registration Token**: エージェントが新しいクラスターを登録するために使用するトークンです。