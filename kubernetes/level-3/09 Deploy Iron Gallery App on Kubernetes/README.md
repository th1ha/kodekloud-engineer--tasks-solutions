# Deploy Iron Gallery App on Kubernetes

1. Create a namespace **iron-namespace-nautilus**

2. Create a deployment **iron-gallery-deployment-nautilus** for **iron gallery** under the same namespace you created.
  -  Labels **run** should be **iron-gallery**.
  - Replicas count should be **1**.
  - Selector's matchLabels **run** should be **iron-gallery**.
  - Template labels **run** should be **iron-gallery** under metadata.
  - The container should be named as **iron-gallery-container-nautilus**, use **kodekloud/irongallery:2.0** image ( use exact image name / tag ).
  - Resources limits for memory should be **100Mi** and for CPU should be **50m**.
  - First volumeMount name should be **config**, its mountPath should be **/usr/share/nginx/html/data**.
  - Second volumeMount name should be **images**, its mountPath should be **/usr/share/nginx/html/uploads**.
  - First volume name should be **config** and give it **emptyDir** and second volume name should be **images**, also give it **emptyDir**.

3. Create a deployment **iron-db-deployment-nautilus** for **iron db** under the same namespace.
  - Labels **db** should be **mariadb**.
  - Replicas count should be **1**.
  - Selector's matchLabels **db** should be **mariadb**.
  - Template labels **db** should be **mariadb** under metadata.
  - The container name should be **iron-db-container-nautilus**, use **kodekloud/irondb:2.0** image ( use exact image name / tag ).
  - Define environment, set **MYSQL_DATABASE** its value should be **database_apache**, set **MYSQL_ROOT_PASSWORD** and **MYSQL_PASSWORD** value should be with some complex passwords for DB connections, and **MYSQL_USER** value should be any custom user ( except root ).
  - Volume mount name should be **db** and its mountPath should be **/var/lib/mysql**. Volume name should be **db** and give it an **emptyDir**.

4. Create a service for **iron db** which should be named **iron-db-service-nautilus** under the same namespace. Configure spec as selector's db should be **mariadb**. Protocol should be **TCP**, port and targetPort should be **3306** and its type should be **ClusterIP**.

5. Create a service for **iron gallery** which should be named **iron-gallery-service-nautilus** under the same namespace. Configure spec as selector's run should be **iron-gallery**. Protocol should be **TCP**, port and targetPort should be **80**, nodePort should be **32678** and its type should be **NodePort**.


- [Doc - container resources example](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#example-1)
- [Doc - emptyDir example](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir-configuration-example)
- [Doc - env variable for a container](https://kubernetes.io/docs/tasks/inject-data-application/define-environment-variable-container/#define-an-environment-variable-for-a-container)
---

<details>
<summary>steps</summary>

  #### create a namespace
  ```bash
  kubectl create ns iron-namespace-nautilus
  ```

  #### change the default namespace
  ```bash
  kubectl config set-context --current --namespace iron-namespace-nautilus

  kubectl config get-contexts
  ```
  - output
    ```
    CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
    *         kind-kodekloud   kind-kodekloud   kind-kodekloud   iron-namespace-nautilus
    ```

  #### create deployment for app
  ```bash
  kubectl create deployment iron-gallery-deployment-nautilus --image kodekloud/irongallery:2.0 --replicas 1 --dry-run=client -oyaml  > iron-deployment.yaml
  ```
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      run: iron-gallery
      app: iron-gallery-deployment-nautilus
    name: iron-gallery-deployment-nautilus
  spec:
    replicas: 1
    selector:
      matchLabels:
        run: iron-gallery
        app: iron-gallery-deployment-nautilus
    strategy: {}
    template:
      metadata:
        labels:
          run: iron-gallery
          app: iron-gallery-deployment-nautilus
      spec:
        volumes:
        - name: config
          emptyDir: {}
        - name: images
          emptyDir: {}
        containers:
        - image: kodekloud/irongallery:2.0
          name: iron-gallery-container-nautilus
          ports:
          - containerPort: 80
          resources:
            limits:
              memory: "100Mi"
              cpu: "50m"
          volumeMounts:
            - name: config
              mountPath: /usr/share/nginx/html/data
            - name: images
              mountPath: /usr/share/nginx/html/uploads
  ```

  #### crreate deployment for db
  ```bash
  kubectl create deployment iron-db-deployment-nautilus --image kodekloud/irondb:2.0 --replicas 1 --port 3306 --dry-run=client -oyaml > iron-db.yaml
  ```
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      db: mariadb
      app: iron-db-deployment-nautilus
    name: iron-db-deployment-nautilus
  spec:
    replicas: 1
    selector:
      matchLabels:
        db: mariadb
        app: iron-db-deployment-nautilus
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          db: mariadb
          app: iron-db-deployment-nautilus
      spec:
        volumes:
        - name: db
          emptyDir: {}
        containers:
        - image: kodekloud/irondb:2.0
          name: iron-db-container-nautilus
          ports:
          - containerPort: 3306
          env:
            - name: MYSQL_DATABASE
              value: "database_apache"
            - name: MYSQL_ROOT_PASSWORD
              value: "ironrootpass123"
            - name: MYSQL_PASSWORD
              value: "ironpass123"
            - name: MYSQL_USER
              value: "iron"
          volumeMounts:
            - name: db
              mountPath: /var/lib/mysql
  ```

  #### kubectl apply -f iron-deployment.yaml -f iron-db.yaml 
    deployment.apps/iron-gallery-deployment-datacenter created
    deployment.apps/iron-db-deployment-datacenter created

  #### kubectl get all
    NAME                                                    READY   STATUS    RESTARTS   AGE
    pod/iron-db-deployment-nautilus-76f5954bd8-47mqh        1/1     Running   0          41s
    pod/iron-gallery-deployment-nautilus-556f477cf4-mhs52   1/1     Running   0          41s

    NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/iron-db-deployment-nautilus        1/1     1            1           41s
    deployment.apps/iron-gallery-deployment-nautilus   1/1     1            1           41s

    NAME                                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/iron-db-deployment-nautilus-76f5954bd8        1         1         1       41s
    replicaset.apps/iron-gallery-deployment-nautilus-556f477cf4   1         1         1       41s

  #### Create service
  ```bash
  kubectl expose deployment iron-db-deployment-nautilus --name iron-db-service-nautilus --port 3306 --target-port 3306 --protocol TCP

  kubectl expose deployment iron-gallery-deployment-nautilus --name iron-gallery-service-nautilus --type NodePort --port 80 --target-port 80 --protocol TCP
  ```

  #### change the NodePort port
  ```bash
  kubectl edit svc iron-gallery-service-nautilus
  ```
  - output
    ```
      ports:
      - nodePort: 31277
      ---
      ports:
      - nodePort: 32678
    ```
</details>