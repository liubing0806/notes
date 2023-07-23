```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: redis

  namespace: yucekj

spec:

  replicas: 1

  selector:

    matchLabels:

      company: yucekj.com

      app: cloud-eyes

      tier: redis

  template:

    metadata:

      labels:

        company: yucekj.com

        app: datacube

        tier: redis

    spec:

      affinity:

        nodeAffinity:

          requiredDuringSchedulingIgnoredDuringExecution:

            nodeSelectorTerms:

            - matchExpressions:

              - key: apps.k8s.cfuture/cloud-eyes

                operator: Exists

      containers:

      - name: redis-container

        image: yucekj.com/redis:v6.0.16

        ports:

        - containerPort: 6379

        resources:

          requests:

            cpu: 100m

            memory: 2048Mi

          limits:

            cpu: 500m

            memory: 2048Mi

---

apiVersion: v1

kind: Service

metadata:

  namespace: yucekj

  name: redis-svc

spec:

  type: ClusterIP

  selector:

    company: yucekj.com

    app: cloud-eyes

    tier: redis

  ports:

  - name: client

    port: 6379
```