```yaml
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
```