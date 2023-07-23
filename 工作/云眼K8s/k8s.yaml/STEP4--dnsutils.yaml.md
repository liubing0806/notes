```yaml
apiVersion: v1

kind: Pod

metadata:

  name: dnsutils

  namespace: yucekj

spec:

  affinity:

    nodeAffinity:

      requiredDuringSchedulingIgnoredDuringExecution:

        nodeSelectorTerms:

        - matchExpressions:
        ## 节点亲和性，以便在调度时将Pod调度到具有键`apps.k8s.cfuture/cloud-eyes`的节点上。

          - key: apps.k8s.cfuture/cloud-eyes

            operator: Exists

  containers:

  - name: dnsutils

    image: mydlqclub/dnsutils:1.3

    imagePullPolicy: IfNotPresent

    command: ["sleep","360000"]

    resources:

      limits:

        cpu: 100m

        memory: 32Mi
```