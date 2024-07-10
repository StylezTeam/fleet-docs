---
title: ""
sidebar_label: "fleet-manager"
---
## fleet-manager



```
fleet-manager [flags]
```

### オプション

```
      --debug                             デバッグログを有効にする
      --debug-level int                   デバッグが有効な場合、klog -v=X を設定する
      --disable-gitops                    gitopsコンポーネントを無効にする
      --disable-metrics                   メトリクスを無効にする
  -h, --help                              fleet-managerのヘルプ
      --kubeconfig string                 kubeconfigへのパス。クラスタ外の場合のみ必要。
      --namespace string                  監視するネームスペース (デフォルト "cattle-fleet-system")
      --shard-id string                   特定のシャードIDでラベル付けされたリソースのみを管理する
      --zap-devel                         開発モードのデフォルト設定(encoder=consoleEncoder,logLevel=Debug,stackTraceLevel=Warn)。本番モードのデフォルト設定(encoder=jsonEncoder,logLevel=Info,stackTraceLevel=Error) (デフォルト true)
      --zap-encoder encoder               Zapログのエンコーディング ('json' または 'console' のいずれか)
      --zap-log-level level               Zapのログレベル。'debug', 'info', 'error' のいずれか、またはカスタムデバッグレベルの増加に対応する任意の整数値 > 0
      --zap-stacktrace-level level        スタックトレースがキャプチャされるZapレベル ('info', 'error', 'panic' のいずれか)
      --zap-time-encoding time-encoding   Zapの時間エンコーディング ('epoch', 'millis', 'nano', 'iso8601', 'rfc3339' または 'rfc3339nano' のいずれか)。デフォルトは 'epoch'
```

### 参照

* [fleet-manager agentmanagement](./fleet-manager_agentmanagement)	 - 
* [fleet-manager cleanup](./fleet-manager_cleanup)	 - 
* [fleet-manager gitjob](./fleet-manager_gitjob)	 - 