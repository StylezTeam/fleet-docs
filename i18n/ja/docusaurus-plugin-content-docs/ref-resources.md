# デプロイされたリソースのリスト

RancherにFleetをインストールした後、これらのリソースがアップストリームクラスターに作成されます。

| タイプ  | 名前        | ネームスペース |
| ----- | ----------- | --------- |
| Helmから、初期設定: | | |
| ClusterRole | fleet-controller | 	- |
| ClusterRole | gitjob | 	- |
| ClusterRoleBinding | fleet-controller | 	- |
| ClusterRoleBinding | gitjob-binding | 	- |
| ConfigMap | fleet-controller | 	cattle-fleet-system |
| Deployment | fleet-controller | 	cattle-fleet-system |
| Deployment | gitjob | 	cattle-fleet-system |
| Role | fleet-controller | 	cattle-fleet-system |
| Role | gitjob | 	cattle-fleet-system |
| RoleBinding | fleet-controller | 	cattle-fleet-system |
| RoleBinding | gitjob | 	cattle-fleet-system |
| Service | gitjob | 	cattle-fleet-system |
| ServiceAccount | fleet-controller | 	cattle-fleet-system |
| ServiceAccount | gitjob | 	cattle-fleet-system |
| 生成されたもの: | | |
| clusters.fleet.cattle.io | local | 	fleet-local |
| clusters.provisioning.cattle.io | local | 	fleet-local |
| clusters.management.cattle.io | local | 	- |
| ClusterGroup | 	default |	fleet-local |
| Bundle | fleet-agent-local | 	fleet-local |
| 各登録クラスターごとに: | | |
| clusters.provisioning.cattle.io | | デフォルトではfleet-default |
| clusters.management.cattle.io | 生成済み |		- |
| clusters.fleet.cattle.io | fleet-default |  |
| Bundle | fleet-default |  |
| BundleDeployment | cluster-fleet-local-local-ID | fleet-agent-local

[namespaces]も参照してください。