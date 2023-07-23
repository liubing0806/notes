```yaml
apiVersion: apps/v1

kind: Deployment

metadata:

  name: cloud-eyes

  namespace: yucekj

spec:

  replicas: 1

  selector:

    matchLabels:

      company: yucekj.com

      app: eyes

      tier: cloud-eyes

  template:

    metadata:

      labels:

        company: yucekj.com

        app: eyes

        tier: cloud-eyes

    spec:

      affinity:

        nodeAffinity:

          requiredDuringSchedulingIgnoredDuringExecution:

            nodeSelectorTerms:

            - matchExpressions:

              - key: apps.k8s.cfuture/eyes

                operator: Exists

      containers:

      - name: cloud-eyes-container

        image: yucekj.com/cloud-eyes:v3.9.16

        envFrom:

        - configMapRef:

            name: mysql-config

        - configMapRef:

            name: redis-config

        ports:

        - containerPort: 7001

        volumeMounts:

        - name: logdir

          mountPath: /opt/yuce/cloud-eyes/logs

        resources:

          requests:

            cpu: 200m

            memory: 1024Mi

          limits:

            cpu: 8

            memory: 8192Mi

      volumes:

      - name: logdir

        hostPath:

          path: /tmp/yuce/cloud-eyes-logs

          type: DirectoryOrCreate

---

apiVersion: v1

kind: Service

metadata:

  name: cloud-eyes-svc

  namespace: yucekj

spec:

  type: ClusterIP

  selector:

    company: yucekj.com

    app: eyes

    tier: cloud-eyes

  ports:

  - name: apiport

    port: 7001
```