# クラスターグループの作成

名前空間内のクラスターはクラスターグループにまとめることができます。クラスターグループは本質的には名前付きセレクターです。
クラスターグループの唯一のパラメータはセレクターです。
ある程度の規模に達すると、クラスターグループはクラスターを管理するためのより合理的な方法となります。
クラスターグループは、デプロイメントの集約されたステータスを提供し、ターゲットを管理するためのより簡単な方法を提供する役割を果たします。

クラスターグループは、以下のように `ClusterGroup` リソースを作成することで作成されます。

```yaml
kind: ClusterGroup
apiVersion: fleet.cattle.io/v1alpha1
metadata:
  name: production-group
  namespace: clusters
spec:
  # これはラベルでクラスターを一致させるための標準的な metav1.LabelSelector フォーマットです
  selector:
    matchLabels:
      env: prod
```