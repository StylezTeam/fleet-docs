---
title: ""
sidebar_label: "fleet target"
---
## fleet target

バンドルの利用可能なターゲットを表示

```
fleet target [flags]
```

### オプション

```
  -b, --bundle-file string                バンドルリソースのyamlの場所
  -l, --dump-input-list                   クラスターなどのターゲティングに影響を与えるライブリソースをYAMLとしてダンプ
  -h, --help                              targetのヘルプ
      --kubeconfig string                 kubeconfigへのパス。クラスタ外の場合のみ必要。
  -n, --namespace string                  バンドルのネームスペースを上書き。ターゲティングはこのネームスペースでクラスターを検索。
      --zap-devel                         開発モードのデフォルト(encoder=consoleEncoder,logLevel=Debug,stackTraceLevel=Warn)。本番モードのデフォルト(encoder=jsonEncoder,logLevel=Info,stackTraceLevel=Error) (デフォルトはtrue)
      --zap-encoder encoder               Zapログのエンコーディング（'json'または'console'のいずれか）
      --zap-log-level level               Zapのログレベル。'debug'、'info'、'error'のいずれか、または任意の整数値 > 0（増加する詳細度のカスタムデバッグレベルに対応）
      --zap-stacktrace-level level        スタックトレースがキャプチャされるZapレベル（'info'、'error'、'panic'のいずれか）
      --zap-time-encoding time-encoding   Zapの時間エンコーディング（'epoch'、'millis'、'nano'、'iso8601'、'rfc3339'、'rfc3339nano'のいずれか）。デフォルトは'epoch'
```

### 参照

* [fleet](./fleet)	 - 