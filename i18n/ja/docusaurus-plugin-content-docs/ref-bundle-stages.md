# バンドルライフサイクル

バンドルは、Gitからのリソースのオーケストレーションに使用される内部リソースです。GitRepoがスキャンされると、1つ以上のバンドルが生成されます。

Fleetバンドルのライフサイクルを示すために、[multi-cluster/helm](https://github.com/rancher/fleet-examples/tree/master/multi-cluster/helm)をケーススタディとして使用します。

1. ユーザーは、multi-cluster/helmリポジトリを指す[GitRepo](./gitrepo-add.md#create-gitrepo-instance)を作成します。
2. `gitjob-controller`はGitRepoからの変更を同期し、ポーリングまたは[Webhookイベント](./webhook.md)からの変更を検出します。コミットの変更ごとに、`gitjob-controller`はGitリポジトリをクローンし、`fleet.yaml`や他のマニフェストなどのリポジトリの内容を読み取り、Fleetの[バンドル](./cluster-bundles-state.md#bundles)を作成するジョブを作成します。

>**注:** `rancher/tekton-utils`というイメージ名のジョブポッドは、GitRepoと同じネームスペースに配置されます。

3. 次に、`fleet-controller`がバンドルからの変更を同期します。ターゲットに応じて、`fleet-controller`はバンドルとターゲットクラスターの組み合わせである`BundleDeployment`リソースを作成します。
4. `fleet-agent`はFleetコントロールプレーンから`BundleDeployment`を取得します。エージェントは、`BundleDeployment`から下流のクラスターにバンドルマニフェストを[Helmチャート](https://helm.sh/docs/intro/install/)としてデプロイします。
5. `fleet-agent`はアプリケーションバンドルを監視し続け、次の順序でステータスを報告します: bundledeployment > bundle > GitRepo > cluster。

この図は、バンドルがデプロイされるまでの異なるレンダリングステージを示しています。

![Bundle Stages](/img/FleetBundleStages.svg)

## CLIを使用したバンドルライフサイクルの検証

いくつかのFleet CLIコマンドは、バンドルのデバッグに役立ちます。

### fleet apply

[Apply](./cli/fleet-cli/fleet_apply.md)は、Helmチャート、マニフェスト、またはkustomizeフォルダーなどのKubernetesリソースを含むフォルダーをFleetバンドルリソースにレンダリングします。

```
git clone https://github.com/rancher/fleet-test-data
cd fleet-test-data
fleet apply -n fleet-local -o bundle.yaml testbundle simple-chart/
```

`fleet apply`を使用してバンドルを作成する方法の詳細は、[バンドルのセクション](https://fleet.rancher.io/bundle-add)にあります。

### fleet target

[Target](./cli/fleet-cli/fleet_target.md)は、ファイルからバンドルを読み取り、ライブクラスターと連携して`bundledeployment`および`content`リソースを出力します。これらはfleetcontrollerが作成するものです。ネームスペースを引数として受け取り、そのネームスペース内のクラスターリソースなどを検索できます。また、「ターゲティング」中に使用されるデータ構造をダンプすることもでき、ラベルやクラスター名に関する決定を確認できます。

### fleet deploy

[Deploy](./cli/fleet-cli/fleet_deploy.md)は、`fleet target`の出力またはダンプされたbundledeployment/contentリソースをクラスターにデプロイします。これは、fleet-agentが行うのと同様です。ドライランモードをサポートしており、リソースをインストールする代わりに作成されるリソースを出力します。このコマンドは入力リソースを作成しないため、実行中のfleet-agentがデプロイメントをガベージコレクトする可能性があります。

デプロイコマンドは、エアギャップクラスターにバンドルを持ち込むために使用できます。

### ライフサイクルCLIの例

```
git clone https://github.com/rancher/fleet-test-data
cd fleet-test-data
# applyに関する情報はhttps://fleet.rancher.io/bundle-addを参照
fleet apply -n fleet-local -o bundle.yaml testbundle simple-chart/
fleet target --bundle-file bundle.yaml --list-inputs  > bd.yaml
fleet deploy --input-file bd.yaml --dry-run
```