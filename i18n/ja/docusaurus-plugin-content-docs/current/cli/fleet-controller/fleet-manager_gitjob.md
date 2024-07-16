---
title: ""
sidebar_label: "fleet-manager gitjob"
---
## fleet-manager gitjob



```
fleet-manager gitjob [flags]
```

### オプション

```
      --debug                         デバッグログを有効にする
      --debug-level int               デバッグが有効な場合、klog -v=X を設定する
      --gitjob-image string           生成されるジョブで使用されるgitjobイメージ。 (デフォルト "rancher/fleet:dev")
  -h, --help                          gitjobのヘルプ
      --kubeconfig string             Kubeconfigファイル
      --leader-elect                  コントローラーマネージャーのリーダー選出を有効にする。これを有効にすると、アクティブなコントローラーマネージャーが1つだけになることを保証します。
      --listen string                 Webhookがリッスンするポート。 (デフォルト ":8080")
      --metrics-bind-address string   メトリックエンドポイントがバインドするアドレス。 (デフォルト ":8081")
      --namespace string              監視するネームスペース (デフォルト "cattle-fleet-system")
```

### 親コマンドから継承されたオプション

```
      --disable-gitops    gitopsコンポーネントを無効にする
      --disable-metrics   メトリクスを無効にする
      --shard-id string   特定のシャードIDでラベル付けされたリソースのみを管理する
```

### 関連項目

* [fleet-manager](./fleet-manager)	 - 