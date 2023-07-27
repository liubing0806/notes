``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yucekj
---

apiVersion: v1

kind: ConfigMap

  

---

apiVersion: v1

kind: ConfigMap

metadata:

  name: redis-config

  namespace: yucekj

data:

  REDIS_HOST: redis-svc.yucekj

  REDIS_PORT: "6379"

  #REDIS_PASSWD: no password supported here.

  

---

# modify your mysql configuration here.

apiVersion: v1

kind: ConfigMap

metadata:

  name: mysql-config

  namespace: yucekj

data:

  MYSQL_HOST: mysql-svc.yucekj

  MYSQL_PORT: "3306"

  MYSQL_USERNAME: root

  MYSQL_DBNAME: Sirius0616
---
apiVersion: v1

kind: ConfigMap

metadata:

  name: nginx-configuration

  namespace: yucekj

data:

  cloud-eyes.conf: |

    server {

        listen 80;

        client_max_body_size 5000M;

        proxy_connect_timeout 300;

        proxy_send_timeout 600;

        proxy_read_timeout 600;

  

        client_header_buffer_size 512k;

        large_client_header_buffers 4 512k;

  

        gzip on;

        gzip_min_length 1k;

        gzip_buffers 4 16k;

        gzip_http_version 1.1;

        gzip_comp_level 9;

        gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/javascript application/json;

        gzip_disable "MSIE [1-6]\.";

        gzip_vary on;

  

        location / {

            gzip on;

            if ($request_filename ~* ^.*?.(html|htm)$) {

                add_header Cache-Control "no-cache";

            }

            if (!-e $request_filename) {

                add_header Cache-Control "no-cache";

            }

  

            alias /opt/yuce/cloud-eyes/dist/;

            try_files $uri /index.html;

            index  index.html index.htm;

        }

        location /api/ {

            if ($request_method = 'OPTIONS') {

                add_header 'Access-Control-Allow-Origin' '*';

                add_header 'Access-Control-Allow-Credentials' 'true';

                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';

                add_header 'Access-Control-Allow-Headers' 'x-authorization,DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';

                add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';

                add_header 'Access-Control-Max-Age' 1728000;

                add_header 'Content-Type' 'text/plain; charset=utf-8';

                add_header 'Content-Length' 0;

                return 204;

            }

            proxy_pass http://cloud-eyes-svc.yucekj:7001/;

        }

    }

---
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
---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  namespace: yucekj
  labels:
    pv: pv-test
spec:
  accessModes:
    - ReadWriteOnce
  capacity:
    storage: 10Gi
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs
  nfs:
    path: /root/nfs_root
    server: 10.0.10.120

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
  namespace: yucekj
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs
  selector:
    matchLabels:
      pv: pv-test
---
apiVersion: apps/v1

kind: Deployment

metadata:

  name: mysql

  namespace: yucekj

  labels:

    app: mysql

spec:

  replicas: 1

  selector:

    matchLabels:

      app: mysql

  strategy:

    type: RollingUpdate

  template:

    metadata:

      labels:

        app: mysql

    spec:

      containers:

        - name: mysql

          image: mysql:5.7.33

          ports:

            - containerPort: 3306

          env:

            - name: MYSQL_ROOT_PASSWORD

              value: "Sirius0616"

          volumeMounts:

            - name: mysql-data

              mountPath: /var/lib/mysql

        volumes:

        - name: mysql-data

          persistentVolumeClaim:

            claimName: mysql-pvc

        - name: time-zone

          hostPath:

            path: /etc/localtime

---

apiVersion: v1

kind: Service

metadata:

  name: mysql-svc

  namespace: yucekj

  labels:

    name: mysql

spec:

  type: NodePort

  ports:

    - port: 3306

      targetPort: 3306

      nodePort: 30036

  selector:

    app: mysql
---
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
----
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
---
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