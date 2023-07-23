```yaml

apiVersion: apps/v1

kind: Deployment

metadata:

  name: frontend

  namespace: yucekj

spec:

  replicas: 1

  selector:

    matchLabels:

      company: yucekj.com

      app: cloud-eyes

      tier: frontend

  template:

    metadata:

      labels:

        company: yucekj.com

        app: cloud-eyes

        tier: frontend

    spec:

      affinity:

        nodeAffinity:

          requiredDuringSchedulingIgnoredDuringExecution:

            nodeSelectorTerms:

            - matchExpressions:
            ## 定义了节点亲和性，以便在调度时将Pod调度到具有键`apps.k8s.cfuture/cloud-eyes`的节点上。

              - key: apps.k8s.cfuture/cloud-eyes

                operator: Exists

      containers:

      - name: frontend-container

        image: cube-eyes-front:v3.9.16

        command: [ "/bin/bash", "-c", "/opt/yuce/nginx/sbin/nginx  && while true; do sleep 3600;done" ]

        ports:

        - containerPort: 80

        volumeMounts:

        - name: nginx-conf

          mountPath: /opt/yuce/nginx/conf/conf.d

        resources:

          requests:

            cpu: 100m

            memory: 256Mi

          limits:

            cpu: 1

            memory: 1024Mi

      volumes:

      - name: nginx-conf

        configMap:

          name: nginx-configuration

---

apiVersion: v1

kind: Service

metadata:

  namespace: yucekj

  name: frontend-svc

spec:

  type: NodePort

  selector:
  ### 意味着该Service将选择所有具有这些标签的Pod，并通过端口80将其暴露给集群外部，节点端口为30080。

    company: yucekj.com

    app: cloud-eyes

    tier: frontend

  ports:

  - protocol: TCP

    port: 80

    targetPort: 80

    nodePort: 30080

```