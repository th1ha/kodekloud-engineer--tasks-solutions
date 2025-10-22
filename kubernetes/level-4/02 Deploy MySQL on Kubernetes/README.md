# Deploy MySQL on Kubernetes

1. Create a PersistentVolume **mysql-pv**, its capacity should be **250Mi**, set other parameters as per your preference.

2. Create a PersistentVolumeClaim to request this PersistentVolume storage. Name it as **mysql-pv-claim** and request a **250Mi** of storage. Set other parameters as per your preference.

3. Create a deployment named **mysql-deployment**, use any mysql image as per your preference. Mount the PersistentVolume at mount path **/var/lib/mysql**.

4. Create a **NodePort** type service named **mysql** and set nodePort to **30007**.

5. Create a secret named **mysql-root-pass** having a key pair value, where key is **password** and its value is **YUIidhb667**, create another secret named **mysql-user-pass** having some key pair values, where frist key is **username** and its value is **kodekloud_aim**, second key is **password** and value is **YchZHRcLkL**, create one more secret named **mysql-db-url**, key name is **database** and value is **kodekloud_db2**

6. Define some Environment variables within the container:
  - **name: MYSQL_ROOT_PASSWORD**, should pick value from secretKeyRef **name: mysql-root-pass** and **key: password**
  - **name: MYSQL_DATABASE**, should pick value from secretKeyRef **name: mysql-db-url** and **key: database**
  - **name: MYSQL_USER**, should pick value from secretKeyRef **name: mysql-user-pass** key **key: username**
  - **name: MYSQL_PASSWORD**, should pick value from secretKeyRef **name: mysql-user-pass** and **key: password**

- [Doc - storage](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)
- [Doc - secret](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-a-container-environment-variable-with-data-from-a-single-secret)
---


<details>
<summary>pv-pvc.yaml</summary>

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: "/mysql-data"

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi
```
</details>

<details>
<summary>steps</summary>

  #### create a storage
  ```bash
  kubectl apply -f pv-pvc.yaml
  ```

  #### kubectl get pv,pvc
    NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
    persistentvolume/mysql-pv   250Mi      RWO            Retain           Bound    default/mysql-pv-claim   manual                  53s

    NAME                                   STATUS   VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    persistentvolumeclaim/mysql-pv-claim   Bound    mysql-pv   250Mi      RWO            manual         53s

  #### create a secret
  ```bash
  kubectl create secret generic mysql-root-pass --from-literal=password=YUIidhb667
  kubectl create secret generic mysql-user-pass --from-literal=username=kodekloud_aim --from-literal=password=YchZHRcLkL
  kubectl create secret generic mysql-db-url --from-literal=database=kodekloud_db2
  ```

  #### Create a deployment and service
  ```bash
  kubectl create deployment mysql-deployment --port 3306 --image mysql:latest --dry-run=client -oyaml > mysql.yaml

  kubectl create service nodeport mysql --tcp=3306:3306 --node-port=30007 --dry-run=client -oyaml >> mysql.yaml
  ```
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: mysql-deployment
    name: mysql-deployment
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: mysql-deployment
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: mysql-deployment
      spec:
        volumes:
        - name: mysql-data
          persistentVolumeClaim:
            claimName: mysql-pv-claim
        containers:
        - image: mysql:latest
          name: mysql
          ports:
          - containerPort: 3306
          volumeMounts:
          - name: mysql-data
            mountPath: /var/lib/mysql
          env:
          - name: MYSQL_ROOT_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-root-pass
                key: password
          - name: MYSQL_DATABASE
            valueFrom:
              secretKeyRef:
                name: mysql-db-url
                key: database
          - name: MYSQL_USER
            valueFrom:
              secretKeyRef:
                name: mysql-user-pass
                key: username
          - name: MYSQL_PASSWORD
            valueFrom:
              secretKeyRef:
                name: mysql-user-pass
                key: password
  ---
  apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: null
    labels:
      app: mysql-deployment
    name: mysql
  spec:
    ports:
    - name: 3306-3306
      nodePort: 30007
      port: 3306
      protocol: TCP
      targetPort: 3306
    selector:
      app: mysql-deployment
    type: NodePort
  status:
    loadBalancer: {}
  ```

  #### kubectl get all
    NAME                                    READY   STATUS    RESTARTS   AGE
    pod/mysql-deployment-568f88bc5d-b4s5q   1/1     Running   0          37s

    NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
    service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          33m
    service/mysql        NodePort    10.96.76.206   <none>        3306:30007/TCP   37s

    NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/mysql-deployment   1/1     1            1           37s

    NAME                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/mysql-deployment-568f88bc5d   1         1         1       37s


  #### Testing
  ```bash
  kubectl exec -it mysql-deployment-568f88bc5d-b4s5q -- sh
  
  sh-5.1# mysql -u kodekloud_aim -p"YchZHRcLkL"

  mysql> show databases;
  ```
</details>