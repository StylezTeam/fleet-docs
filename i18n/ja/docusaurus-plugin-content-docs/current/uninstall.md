# アンインストール

Fleetは2つのHelmチャートとしてパッケージ化されているため、適切なHelmチャートをアンインストールすることでアンインストールが完了します。Fleetをアンインストールするには、以下の2つのコマンドを実行します:

```shell
helm -n cattle-fleet-system uninstall fleet
helm -n cattle-fleet-system uninstall fleet-crd
```

:::caution
CRDをアンインストールすると、すべてのデプロイされたワークロードが削除されます。
:::