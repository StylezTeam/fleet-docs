# 変更されたGitRepoを無視するDiffの生成

Rancherの継続的デリバリーはfleetによって動作します。ユーザーがGitRepo CRを追加すると、継続的デリバリーは関連するfleetバンドルを作成します。

これらのバンドルには、クラスタエクスプローラー（ダッシュボードUI）に移動し、`Bundles`セクションを選択することでアクセスできます。

バンドルされたチャートには、実行時に修正されるオブジェクトが含まれている場合があります。例えば、ValidatingWebhookConfigurationでは`caBundle`が空であり、CA証明書がクラスタによって注入されます。

これにより、バンドルおよび関連するGitRepoのステータスが「Modified」と報告されます。

![](/img/ModifiedGitRepo.png)

関連するバンドル
![](/img/ModifiedBundle.png)

Fleetバンドルは、カスタムの[jsonPointerパッチ](http://jsonpatch.com/)を指定する機能をサポートしています。

このパッチを使用すると、ユーザーはfleetにオブジェクトの変更を無視するよう指示できます。

## 簡単な例

この簡単な例では、バンドルの差分を適用するServiceとConfigMapを作成します。

https://github.com/rancher/fleet-test-data/tree/master/bundle-diffs

## Gatekeeperの例

この例では、継続的デリバリーを使用してopa-gatekeeperをクラスタにデプロイしようとしています。

opa GitRepoに関連するopa-gatekeeperバンドルは変更された状態です。

GitRepo CRの各パスには、関連するBundle CRがあります。ユーザーはバンドルを表示し、バンドルステータスに必要な関連する差分を確認できます。

私たちの場合、検出された違いは次のとおりです：

```yaml
  summary:
    desiredReady: 1
    modified: 1
    nonReadyResources:
    - bundleState: Modified
      modifiedStatus:
      - apiVersion: admissionregistration.k8s.io/v1
        kind: ValidatingWebhookConfiguration
        name: gatekeeper-validating-webhook-configuration
        patch: '{"$setElementOrder/webhooks":[{"name":"validation.gatekeeper.sh"},{"name":"check-ignore-label.gatekeeper.sh"}],"webhooks":[{"clientConfig":{"caBundle":"Cg=="},"name":"validation.gatekeeper.sh","rules":[{"apiGroups":["*"],"apiVersions":["*"],"operations":["CREATE","UPDATE"],"resources":["*"]}]},{"clientConfig":{"caBundle":"Cg=="},"name":"check-ignore-label.gatekeeper.sh","rules":[{"apiGroups":[""],"apiVersions":["*"],"operations":["CREATE","UPDATE"],"resources":["namespaces"]}]}]}'
      - apiVersion: apps/v1
        kind: Deployment
        name: gatekeeper-audit
        namespace: cattle-gatekeeper-system
        patch: '{"spec":{"template":{"spec":{"$setElementOrder/containers":[{"name":"manager"}],"containers":[{"name":"manager","resources":{"limits":{"cpu":"1000m"}}}],"tolerations":[]}}}}'
      - apiVersion: apps/v1
        kind: Deployment
        name: gatekeeper-controller-manager
        namespace: cattle-gatekeeper-system
        patch: '{"spec":{"template":{"spec":{"$setElementOrder/containers":[{"name":"manager"}],"containers":[{"name":"manager","resources":{"limits":{"cpu":"1000m"}}}],"tolerations":[]}}}}'
```

この要約に基づいて、パッチが必要なオブジェクトが3つあります。

これらを一つずつ見ていきます。

### 1. ValidatingWebhookConfiguration:
gatekeeper-validating-webhook-configurationのvalidating webhookには、specに2つのValidatingWebhooksがあります。

フィールド内の複数の要素にパッチが必要な場合、そのパッチは`$setElementOrder/ELEMENTNAME`として参照されます。

この情報から、問題の2つのValidatingWebhooksは次のとおりです：

```
  "$setElementOrder/webhooks": [
    {
      "name": "validation.gatekeeper.sh"
    },
    {
      "name": "check-ignore-label.gatekeeper.sh"
    }
  ],
```

各ValidatingWebhook内で無視する必要があるフィールドは次のとおりです：

```
    {
      "clientConfig": {
        "caBundle": "Cg=="
      },
      "name": "validation.gatekeeper.sh",
      "rules": [
        {
          "apiGroups": [
            "*"
          ],
          "apiVersions": [
            "*"
          ],
          "operations": [
            "CREATE",
            "UPDATE"
          ],
          "resources": [
            "*"
          ]
        }
      ]
    },
 ```

 および

 ```
     {
      "clientConfig": {
        "caBundle": "Cg=="
      },
      "name": "check-ignore-label.gatekeeper.sh",
      "rules": [
        {
          "apiGroups": [
            ""
          ],
          "apiVersions": [
            "*"
          ],
          "operations": [
            "CREATE",
            "UPDATE"
          ],
          "resources": [
            "namespaces"
          ]
        }
      ]
    }
```

要約すると、パッチ仕様で無視する必要があるフィールドは`rules`と`clientConfig.caBundle`です。

ValidatingWebhookConfigurationのspec内のwebhookフィールドは配列なので、要素をインデックス値で指定する必要があります。

![](/img/WebhookConfigurationSpec.png)

この情報に基づいて、差分パッチは次のようになります：

```yaml
  - apiVersion: admissionregistration.k8s.io/v1
    kind: ValidatingWebhookConfiguration
    name: gatekeeper-validating-webhook-configuration
    operations:
    - {"op": "remove", "path":"/webhooks/0/clientConfig/caBundle"}
    - {"op": "remove", "path":"/webhooks/0/rules"}
    - {"op": "remove", "path":"/webhooks/1/clientConfig/caBundle"}
    - {"op": "remove", "path":"/webhooks/1/rules"}
```

### 2. Deployment gatekeeper-controller-manager:
gatekeeper-controller-managerのデプロイメントは、CPU制限とトレランスが適用されているため（実際のバンドルにはない）、変更されています。

```
{
  "spec": {
    "template": {
      "spec": {
        "$setElementOrder/containers": [
          {
            "name": "manager"
          }
        ],
        "containers": [
          {
            "name": "manager",
            "resources": {
              "limits": {
                "cpu": "1000m"
              }
            }
          }
        ],
        "tolerations": []
      }
    }
  }
}
```

この場合、デプロイメントのコンテナspecには1つのコンテナしかなく、そのコンテナにはCPU制限とトレランスが追加されています。

この情報に基づいて、差分パッチは次のようになります：
```yaml
  - apiVersion: apps/v1
    kind: Deployment
    name: gatekeeper-controller-manager
    namespace: cattle-gatekeeper-system
    operations:
    - {"op": "remove", "path": "/spec/template/spec/containers/0/resources/limits/cpu"}
    - {"op": "remove", "path": "/spec/template/spec/tolerations"}
```

### 3. Deployment gatekeeper-audit:
gatekeeper-auditのデプロイメントも、gatekeeper-controller-managerと同様に、追加のCPU制限とトレランスが適用されています。

```
{
  "spec": {
    "template": {
      "spec": {
        "$setElementOrder/containers": [
          {
            "name": "manager"
          }
        ],
        "containers": [
          {
            "name": "manager",
            "resources": {
              "limits": {
                "cpu": "1000m"
              }
            }
          }
        ],
        "tolerations": []
      }
    }
  }
}
```

gatekeeper-controller-managerと同様に、デプロイメントのコンテナspecには1つのコンテナしかなく、CPU制限とトレランスが追加されています。

この情報に基づいて、差分パッチは次のようになります：
```yaml
  - apiVersion: apps/v1
    kind: Deployment
    name: gatekeeper-audit
    namespace: cattle-gatekeeper-system
    operations:
    - {"op": "remove", "path": "/spec/template/spec/containers/0/resources/limits/cpu"}
    - {"op": "remove", "path": "/spec/template/spec/tolerations"}
```

### すべてをまとめる
これらのパッチを次のようにまとめることができます：

```yaml
diff:
  comparePatches:
  - apiVersion: apps/v1
    kind: Deployment
    name: gatekeeper-audit
    namespace: cattle-gatekeeper-system
    operations:
    - {"op": "remove", "path": "/spec/template/spec/containers/0/resources/limits/cpu"}
    - {"op": "remove", "path": "/spec/template/spec/tolerations"}
  - apiVersion: apps/v1
    kind: Deployment
    name: gatekeeper-controller-manager
    namespace: cattle-gatekeeper-system
    operations:
    - {"op": "remove", "path": "/spec/template/spec/containers/0/resources/limits/cpu"}
    - {"op": "remove", "path": "/spec/template/spec/tolerations"}
  - apiVersion: admissionregistration.k8s.io/v1
    kind: ValidatingWebhookConfiguration
    name: gatekeeper-validating-webhook-configuration
    operations:
    - {"op": "remove", "path":"/webhooks/0/clientConfig/caBundle"}
    - {"op": "remove", "path":"/webhooks/0/rules"}
    - {"op": "remove", "path":"/webhooks/1/clientConfig/caBundle"}
    - {"op": "remove", "path":"/webhooks/1/rules"}
```

これらをバンドルに直接追加してテストし、同じ内容をGitRepoの`fleet.yaml`にコミットすることができます。

これらが追加されると、GitRepoはデプロイされ、「Active」ステータスになるはずです。