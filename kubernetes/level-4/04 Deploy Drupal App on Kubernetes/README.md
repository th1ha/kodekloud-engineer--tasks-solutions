# Deploy Drupal App on Kubernetes

1. Configure a persistent volume **drupal-mysql-pv** with **hostPath = /drupal-mysql-data** (**/drupal-mysql-data** directory already exists on the worker Node i.e jump host), **5Gi** of storage and **ReadWriteOnce** access mode.

2. Configure one PersistentVolumeClaim named **drupal-mysql-pvc** with storage request of **3Gi** and **ReadWriteOnce** access mode.

3. Create a deployment **drupal-mysql** with **1** replica, use **mysql:5.7** image. Mount the claimed PVC at **/var/lib/mysql**.

4. Create a deployment **drupal** with **1** replica and use **drupal:8.6** image.
    - Create a **NodePort** type service which should be named as **drupal-service** and nodePort should be **30095**.

5. Create a service **drupal-mysql-service** to expose mysql deployment on port **3306**.

6. Set rest of the configration for deployments, services, secrets etc as per your preferences. At the end you should be able to access the Drupal installation page by clicking on **App** button.

- [Doc - Storage ](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
---

<details>
<summary>pv-pvc.yaml</summary>

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: drupal-mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/drupal-mysql-data"
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: drupal-mysql-pvc
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```
</details>

<details>
<summary>steps</summary>

  #### create pv & pvc
  ```bash
  kubectl apply -f pv-pvc.yaml
  ```

  #### create a deployment and service
  ```bash
  kubectl create deployment drupal-mysql --image mysql:5.7 --replicas 1 --dry-run=client -oyaml > drupal-mysql.yaml

  kubectl create service clusterip drupal-mysql-service --tcp 3306:3306 --dry-run=client -oyaml >> drupal-mysql.yaml

  kubectl create deployment drupal --image drupal:8.6 --replicas 1 --port 80 --dry-run=client -oyaml > drupal.yaml

  kubectl create service nodeport drupal-service --tcp 80:80 --node-port 30095 --dry-run=client -oyaml >> drupal.yaml

  kubectl apply -f drupal-mysql.yaml -f drupal.yaml
  ```

  #### drupal-mysql deployment
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: drupal-mysql
    name: drupal-mysql
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: drupal-mysql
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: drupal-mysql
      spec:
        volumes:
        - name: task-pv-storage
          persistentVolumeClaim:
            claimName: drupal-mysql-pvc
        containers:
        - image: mysql:5.7
          name: mysql
          ports:
          - containerPort: 3306
          env:
            - name: MYSQL_DATABASE
              value: drupal
            - name: MYSQL_USER
              value: drupal
            - name: MYSQL_PASSWORD
              value: drupal123
            - name: MYSQL_ROOT_PASSWORD
              value: root123
          volumeMounts:
          - mountPath: "/var/lib/mysql"
            name: task-pv-storage
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: drupal-mysql-service
  spec:
    ports:
    - name: 3306-3306
      port: 3306
      protocol: TCP
      targetPort: 3306
    selector:
      app: drupal-mysql
    type: ClusterIP
  ```

  #### drupal deployment
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: drupal
    name: drupal
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: drupal
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: drupal
      spec:
        containers:
        - image: drupal:8.6
          name: drupal
          ports:
          - containerPort: 80
          env:
            - name: DRUPAL_DATABASE_HOST
              value: drupal-mysql-service
            - name: DRUPAL_DATABASE_NAME
              value: drupal
            - name: DRUPAL_DATABASE_USER
              value: drupal
            - name: DRUPAL_DATABASE_PASSWORD
              value: drupal123
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: drupal-service
  spec:
    ports:
    - name: 80-80
      nodePort: 30095
      port: 80
      protocol: TCP
      targetPort: 80
    selector:
      app: drupal
    type: NodePort
  ```
</details>