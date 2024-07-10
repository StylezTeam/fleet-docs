---
title: ""
sidebar_label: "fleet-agent"
---
## fleet-agent

```
fleet-agent [flags]
```

### オプション

```
      --agent-scope string                エージェントのバンドルID名をスコープするために使用される識別子。通常は名前空間と同じ
      --debug                             デバッグログを有効にする
      --debug-level int                   デバッグが有効な場合、klog -v=X を設定
  -h, --help                              fleet-agent のヘルプ
      --kubeconfig string                 kubeconfig へのパス。クラスタ外の場合のみ必要
      --namespace string                  エージェントが実行されるシステム名前空間。例: cattle-fleet-system
      --zap-devel                         開発モードのデフォルト設定(encoder=consoleEncoder,logLevel=Debug,stackTraceLevel=Warn)。本番モードのデフォルト設定(encoder=jsonEncoder,logLevel=Info,stackTraceLevel=Error) (デフォルトは true)
      --zap-encoder encoder               Zap ログのエンコーディング ( 'json' または 'console' のいずれか)
      --zap-log-level level               Zap ログの詳細度を設定。 'debug', 'info', 'error' のいずれか、またはカスタムデバッグレベルの増加する詳細度に対応する任意の整数値 > 0
      --zap-stacktrace-level level        スタックトレースがキャプチャされる Zap レベル ( 'info', 'error', 'panic' のいずれか)
      --zap-time-encoding time-encoding   Zap 時間エンコーディング ( 'epoch', 'millis', 'nano', 'iso8601', 'rfc3339' または 'rfc3339nano' のいずれか)。デフォルトは 'epoch'
```

### 参照

* [fleet-agent clusterstatus](./fleet-agent_clusterstatus) - リソースのステータスを上流クラスタに継続的に報告
* [fleet-agent register](./fleet-agent_register) - エージェントを上流クラスタに登録