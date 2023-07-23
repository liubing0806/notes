```yaml

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

  #MYSQL_PASSWD: no password supported here.

```