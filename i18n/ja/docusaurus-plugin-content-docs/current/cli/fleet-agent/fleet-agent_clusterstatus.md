---
title: ""
sidebar_label: "fleet-agent clusterstatus"
---
## fleet-agent clusterstatus

リソースのステータスを上流クラスターに継続的に報告する

```
fleet-agent clusterstatus [flags]
```

### オプション

```
      --checkin-interval string   クラスターのステータスを投稿する頻度
      --debug                     デバッグログを有効にする
      --debug-level int           デバッグが有効な場合、klog -v=X を設定する
  -h, --help                      clusterstatus のヘルプ
      --kubeconfig string         エージェントのクラスター用の kubeconfig ファイル
      --namespace string          エージェントが実行されるシステムネームスペース、例: cattle-fleet-system
```

### 参照

* [fleet-agent](./fleet-agent)	 -