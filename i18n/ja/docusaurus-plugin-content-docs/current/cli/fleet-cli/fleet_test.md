---
title: ""
sidebar_label: "fleet test"
---
## fleet test

バンドルをターゲットにマッチさせて出力をレンダリングする（非推奨）

```
fleet test [flags]
```

### オプション

```
  -b, --bundle-file string    生のバンドルリソースyamlの場所
  -f, --file string           fleet.yamlの場所
  -g, --group string          マッチさせるクラスターグループ
  -L, --group-label strings   マッチさせるクラスターグループラベル
  -h, --help                  testのヘルプ
  -l, --label strings         マッチさせるクラスターラベル
  -N, --name string           マッチさせるクラスター名
  -q, --quiet                 マッチだけを表示し、リソースを表示しない
  -t, --target string         明示的にマッチさせるターゲット
```

### 参照

* [fleet](./fleet)	 - 