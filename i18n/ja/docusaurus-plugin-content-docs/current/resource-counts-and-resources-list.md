# リソースリスト

このドキュメントは、`Bundles` と `GitRepos` に分類されたデプロイされたリソースを概説しています。

## Bundles

バンドル内のデプロイされたリソースは `status.ResourceKey` にあります。このキーは `bundleDeployments` を通じて実際にデプロイされたリソースを表しています。

## GitRepos

バンドルと同様に、GitRepos 内のデプロイされたリソースは `status.Resources` にリストされています。このリストも `bundleDeployments` から派生しています。

# リソース数

## GitRepos

GitRepos の `status.ResourceCounts` リストは `bundleDeployments` から派生しています。

## クラスター

クラスターでは、`status.ResourceCounts` リストは GitRepos から派生しています。

## クラスターグループ

クラスターグループでは、`status.ResourceCounts` リストも GitRepos から派生しています。