---
title: ""
sidebar_label: "fleet apply"
---
## fleet apply

ディレクトリからバンドルを作成し、それを出力するかクラスターに適用します

```
fleet apply [flags] BUNDLE_NAME PATH...
```

### オプション

```
  -b, --bundle-file string                     生のバンドルリソースyamlの場所
      --cacerts-file string                    Helmリポジトリのカスタムcacertsのパス
      --commit string                          バンドルに割り当てるコミット
  -c, --compress                               すべてのリソースを圧縮するよう強制
      --context string                         認証のためのkubeconfigコンテキスト
      --correct-drift                          Fleet外部からの変更をロールバック
      --correct-drift-force                    ドリフトを修正する際に--forceを使用。リソースが削除され再作成される可能性あり
      --correct-drift-keep-fail-history        失敗したロールバックのためのHelm履歴を保持
      --debug                                  デバッグログを有効にする
      --debug-level int                        デバッグが有効な場合、klog -v=Xを設定
  -f, --file string                            fleet.yamlの場所
      --helm-credentials-by-path-file string   パスのHelm認証情報を含むファイルのパス
      --helm-repo-url-regex string             Helmリポジトリがこの正規表現に一致する場合、Helm認証情報が使用される。この正規表現が空または提供されていない場合、常に認証情報が使用される
  -h, --help                                   applyのヘルプ
      --keep-resources                         GitRepoまたはバンドルが削除された後も作成されたリソースを保持
  -k, --kubeconfig string                      認証のためのkubeconfig
  -l, --label strings                          作成されたバンドルに適用するラベル
  -n, --namespace string                       ネームスペース（デフォルトは "fleet-local"）
  -o, --output string                          ファイルまたは標準出力に内容を出力
      --password-file string                   Helmリポジトリの基本認証パスワードを含むファイルのパス
      --paused                                 一時停止状態でバンドルを作成
  -a, --service-account string                 作成されたバンドルに割り当てるサービスアカウント
      --ssh-privatekey-file string             Helmリポジトリのsshプライベートキーのパス
      --sync-generation int                    デプロイメントを強制同期するために使用される世代番号
      --target-namespace string                このバンドルがこのターゲットネームスペースに行くことを保証
      --targets-file string                    追加のターゲットと制限を追加するためのソース
      --username string                        Helmリポジトリの基本認証ユーザー名
```

### 参照

* [fleet](./fleet)	 - 