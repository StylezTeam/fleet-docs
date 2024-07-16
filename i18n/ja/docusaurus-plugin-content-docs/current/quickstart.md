import {versions} from '@site/src/fleetVersions';
import CodeBlock from '@theme/CodeBlock';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# クイックスタート

![](/img/single-cluster.png)

ドキュメントなんていらない、さっそくこれを実行しよう！

## インストール

FleetはHelmチャートとして配布されています。Helm 3はCLIであり、サーバーサイドコンポーネントはなく、その使用は非常に簡単です。Helm 3 CLIをインストールするには、<a href="https://helm.sh/docs/intro/install">公式インストール手順</a>に従ってください。

:::caution RancherのFleet
RancherにはFleet用の別のHelmチャートがあり、異なるリポジトリを使用します。
:::

<Tabs>
  <TabItem value="linux" label="Linux/Mac" default>
    <CodeBlock language="bash">
    {`brew install helm\n`}
    {`helm repo add fleet https://rancher.github.io/fleet-helm-charts/`}
    </CodeBlock>
  </TabItem>
  <TabItem value="windows" label="Windows" default>
    <CodeBlock language="bash">
    {`choco install kubernetes-helm\n`}
    {`helm repo add fleet https://rancher.github.io/fleet-helm-charts/`}
    </CodeBlock>
  </TabItem>
</Tabs>

Fleet Helmチャートをインストールします（CRDを分離して究極の柔軟性を提供するため、2つあります）。

<CodeBlock language="bash">
{`helm -n cattle-fleet-system install --create-namespace --wait fleet-crd \\
    fleet/fleet-crd\n`}
{`helm -n cattle-fleet-system install --create-namespace --wait fleet \\
    fleet/fleet`}
</CodeBlock>

## 監視するGitリポジトリを追加

`spec.repo`をお好みのGitリポジトリに変更します。デプロイするKubernetesマニフェストファイルは、リポジトリの`/manifests`に配置する必要があります。

```bash
cat > example.yaml << "EOF"
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: sample
  # このネームスペースは特別で、ローカルクラスターに自動的にデプロイされます
  namespace: fleet-local
spec:
  # このリポジトリのすべてがこのクラスターで実行されます。私を信じてくださいね？
  repo: "https://github.com/rancher/fleet-examples"
  paths:
  - simple
EOF

kubectl apply -f example.yaml
```

## ステータスを取得

Fleetが何をしているかのステータスを取得します。

```shell
kubectl -n fleet-local get fleet
```

クラスターに次のようなものが作成されるのが見えるはずです。

```
kubectl get deploy frontend
```
```
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   3/3     3            3           116m
```

楽しんで、[ドキュメント](https://rancher.github.io/fleet)を読んでください。
