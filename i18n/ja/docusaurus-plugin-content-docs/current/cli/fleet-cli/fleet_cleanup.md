---
title: ""
sidebar_label: "fleet cleanup"
---
## fleet cleanup

古いクラスター登録のクリーンアップ

```
fleet cleanup [flags]
```

### オプション

```
      --context string      認証のためのkubeconfigコンテキスト
      --debug               デバッグログを有効にする
      --debug-level int     デバッグが有効な場合、klog -v=Xを設定
      --factor string       削除間の遅延を増加させるファクター (デフォルト: 1.1)
  -h, --help                cleanupのヘルプ
  -k, --kubeconfig string   認証のためのkubeconfig
      --max string          削除間の最大遅延 (デフォルト: 5s)
      --min string          削除間の最小遅延 (デフォルト: 10ms)
  -n, --namespace string    ネームスペース (デフォルト "fleet-local")
```

### 参照

* [fleet](./fleet)	 - 