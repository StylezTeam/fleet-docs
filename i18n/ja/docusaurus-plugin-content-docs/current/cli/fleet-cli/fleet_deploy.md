---
title: ""
sidebar_label: "fleet deploy"
---
## fleet deploy

Helmリリースを作成することにより、バンドルデプロイメント/コンテンツリソースをクラスターにデプロイします。これにより、バンドルデプロイメント/コンテンツリソースが直接クラスターにデプロイされることはありません。

```
fleet deploy [flags]
```

### オプション

```
  -a, --agent-namespace string            エージェントのネームスペースを設定します。通常はcattle-fleet-systemです。設定されている場合、フリートエージェントはHelmリリースをガベージコレクトします。つまり、バンドルデプロイメントが存在しない場合は削除します。
  -d, --dry-run                           デプロイされるリソースを表示しますが、実際にはデプロイしません
  -h, --help                              deployのヘルプを表示します
  -i, --input-file string                 コンテンツとバンドルデプロイメントリソースを含むYAMLファイルの場所
      --kubeconfig string                 kubeconfigのパス。クラスター外の場合のみ必要です。
  -n, --namespace string                  デフォルトのネームスペースを設定します。このネームスペースにHelmチャートをデプロイします。
      --zap-devel                         開発モードのデフォルト設定(encoder=consoleEncoder,logLevel=Debug,stackTraceLevel=Warn)。本番モードのデフォルト設定(encoder=jsonEncoder,logLevel=Info,stackTraceLevel=Error) (デフォルトはtrue)
      --zap-encoder encoder               Zapログのエンコーディング（'json'または'console'のいずれか）
      --zap-log-level level               Zapのログレベルを設定します。'debug', 'info', 'error'のいずれか、または任意の整数値 > 0（増加する詳細度に対応するカスタムデバッグレベル）
      --zap-stacktrace-level level        スタックトレースがキャプチャされるZapレベル（'info', 'error', 'panic'のいずれか）
      --zap-time-encoding time-encoding   Zapの時間エンコーディング（'epoch', 'millis', 'nano', 'iso8601', 'rfc3339'または'rfc3339nano'のいずれか）。デフォルトは'epoch'。
```

### 関連項目

* [fleet](./fleet)	 - 