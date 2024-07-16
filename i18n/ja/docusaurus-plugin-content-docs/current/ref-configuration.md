# 設定

主に内部の設定オプションのリファレンスリスト。

## Helm Charts

Helmチャートは、少なくとも`values.yaml`に示されているデフォルトのオプションを受け入れます：

* https://github.com/rancher/fleet/blob/main/charts/fleet/values.yaml
* https://github.com/rancher/fleet/blob/main/charts/fleet-crd/values.yaml
* https://github.com/rancher/fleet/blob/main/charts/fleet-agent/values.yaml

## 環境変数

コントローラーは以下の環境変数で起動できます：

* `CATTLE_DEV_MODE` - wranglerのデバッグに使用されますが、使用不可
* `FLEET_CLUSTER_ENQUEUE_DELAY` - 準備が整っていないクラスターがチェックされる頻度を調整
* `FLEET_CPU_PPROF_PERIOD` - [パフォーマンスプロファイリング](https://github.com/rancher/fleet/blob/main/docs/performance.md)を有効にするために使用

## 設定

エージェントとフリートマネージャーのクラスター設定。これらを変更すると、完全な再デプロイが発生する可能性があります。

[構造体](https://github.com/rancher/fleet/blob/main/internal/config/config.go#L57)は両方のコンフィグマップで使用されます：

* cattle-fleet-system/fleet-agent
* cattle-fleet-system/fleet-controller

## ラベル

フリートで使用されるラベル：

* `fleet.cattle.io/agent=true` - エージェントのデプロイメントアフィニティ設定のためのNodeSelectorラベル
* `fleet.cattle.io/non-managed-agent` - 管理エージェントバンドルはこのラベルを持つクラスターを対象にしません
* `fleet.cattle.io/repo-name` - Gitリポジトリリソースを参照するためにバンドルで使用
* `fleet.cattle.io/bundle-namespace` - バンドルリソースを参照するためにBundleDeploymentで使用
* `fleet.cattle.io/bundle-name` - バンドルリソースを参照するためにBundleDeploymentで使用
* `fleet.cattle.io/managed=true` - このラベルを持つクラスターのネームスペースはクリーンアップされます。他のリソースもラベルに含まれている場合はクリーンアップされます。Rancherではフリートネームスペースを識別するために使用されます。
* `fleet.cattle.io/bootstrap-token` - 未使用
* `fleet.cattle.io/shard-id=<shard-id>` - フリートコントローラーポッドのシャードID。
* `fleet.cattle.io/shard-default=true` - シャード参照ラベルのないリソースを管理するコントローラーである場合はtrue。
* `fleet.cattle.io/shard-ref=<shard-id>` - Fleetによってリソースに割り当てられたシャードIDを参照し、`GitRepo`から継承され、どのFleetコントローラーデプロイメントがそれらを調整するかを決定します。
    * このラベルが提供されていないか空の値である場合、シャードされていないFleetコントローラーがリソースを処理します。
    * このラベルの値がデプロイされたFleetコントローラーのシャードIDと一致しない場合、リソースは処理されません。

## アノテーション

フリートで使用されるアノテーション：

* `fleet.cattle.io/agent-namespace`
* `fleet.cattle.io/bundle-id`
* `fleet.cattle.io/cluster`, `fleet.cattle.io/cluster-namespace` - クラスター登録ネームスペースとクラスター名を参照するためにクラスターネームスペースで使用
* `fleet.cattle.io/cluster-group`
* `fleet.cattle.io/cluster-registration-namespace`
* `fleet.cattle.io/cluster-registration`
* `fleet.cattle.io/commit`
* `fleet.cattle.io/managed` - 未使用のようです
* `fleet.cattle.io/service-account`

## フリートエージェントの設定

フリートエージェントのためのトレランス、アフィニティ、およびリソースはカスタマイズ可能です。これらのフィールドは[クラスター](https://fleet.rancher.io/ref-crds#clusterspec)を作成する際に提供できます。クラスターの作成方法については[ダウンストリームクラスターの登録](https://fleet.rancher.io/cluster-registration)を参照してください。これらのフィールドが提供されていない場合、デフォルトの設定が使用されます。

リソース制限を変更する場合、フリートエージェントが正常に動作するための制限を確保してください。

フリートをv0.7.0以前のバージョンにダウングレードすると、フリートは組み込みのデフォルトにフォールバックすることに注意してください。エージェントがカスタムアフィニティを持っていた場合、再デプロイされます。フリートのバージョン番号が変更されない場合、再デプロイは即座に行われない可能性があります。