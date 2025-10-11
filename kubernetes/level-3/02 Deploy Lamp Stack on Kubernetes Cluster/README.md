# Deploy Lamp Stack on Kubernetes Cluster

1. Create a config map **php-config** for **php.ini** with **variables_order = "EGPCS"** data.

2. Create a deployment named **lamp-wp**.

3. Create two containers under it. First container must be **httpd-php-container** using image **webdevops/php-apache:alpine-3-php7** and second container must be **mysql-container** from image **mysql:5.6**. Mount **php-config** configmap in httpd container at **/opt/docker/etc/php/php.ini** location.

4. Create kubernetes generic secrets for mysql related values like myql root password, mysql user, mysql password, mysql host and mysql database. Set any values of your choice.

5. Add some environment variables for both containers:
    - **MYSQL_ROOT_PASSWORD**, **MYSQL_DATABASE**, **MYSQL_USER**, **MYSQL_PASSWORD** and **MYSQL_HOST**. Take their values from the secrets you created. Please make sure to use **env** field (do not use **envFrom**) to define the name-value pair of environment variables.

6. Create a node port type service **lamp-service** to expose the web application, nodePort must be **30008**.

7. Create a service for mysql named **mysql-service** and its port must be **3306**.

8. We already have **/tmp/index.php** file on **jump_host** server.
    - Copy this file into httpd container under Apache document root i.e **/app** and replace the dummy values for mysql related variables with the environment variables you have set for mysql related parameters. Please make sure you do not hard code the mysql related details in this file, you must use the environment variables to fetch those values.
    - You must be able to access this **index.php** on node port **30008** at the end, please note that you should see **Connected successfully** message while accessing this page.

- [Doc - Add ConfigMap ddata to a Volume](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#add-configmap-data-to-a-volume)
- [Doc - Using subPath](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath)
- [Doc - Using Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/)
- [Define container environment variables with data from multiple Secrets](https://kubernetes.io/docs/tasks/inject-data-application/distribute-credentials-secure/#define-container-environment-variables-with-data-from-multiple-secrets)


<details>
<summary>steps</summary>

  #### kubectl create cm php-config --from-literal=php.ini='variables_order = "EGPCS"'
    # configmap/php-config created

  #### create secrets
  ```bash
  kubectl create secret generic mysql-secrets \
  --from-literal=root_password=rootpass123 \
  --from-literal=username=wpuser \
  --from-literal=password=wppass123 \
  --from-literal=database=wpdb \
  --from-literal=host=mysql-service
  ```
    # secret/mysql-secrets created

  #### kubectl get secrets mysql-secrets -oyaml
  ```yaml
  apiVersion: v1
  data:
    database: d3BkYg==
    host: bXlzcWwtc2VydmljZQ==
    password: d3BwYXNzMTIz
    root_password: cm9vdHBhc3MxMjM=
    username: d3B1c2Vy
  kind: Secret
  metadata:
    name: mysql-secrets
    namespace: default
  type: Opaque
  ```

  #### kubectl create deployment lamp-wp --image=webdevops/php-apache:alpine-3-php7 --image=mysql:5.6 --dry-run=client -oyaml > lamp-wp.yaml
  - vi lamp-wp.yaml
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app: lamp-wp
    name: lamp-wp
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: lamp-wp
    template:
      metadata:
        labels:
          app: lamp-wp
      spec:
        volumes:
          - name: php-config-volume
            configMap:
              name: php-config
        containers:
        - image: webdevops/php-apache:alpine-3-php7
          name: httpd-php-container
          ports:
            - containerPort: 80
          volumeMounts:
            - mountPath: /opt/docker/etc/php/php.ini
              name: php-config-volume
              subPath: php.ini
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: root_password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: host
        - image: mysql:5.6
          name: mysql-container
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: root_password
            - name: MYSQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: database
            - name: MYSQL_USER
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: username
            - name: MYSQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: password
            - name: MYSQL_HOST
              valueFrom:
                secretKeyRef:
                  name: mysql-secrets
                  key: host
  ```

  #### kubectl apply -f lamp-wp.yaml 
    deployment.apps/lamp-wp created

  #### kubectl expose deployment lamp-wp --name mysql-service --target-port 3306 --port 3306
    service/mysql-service exposed

  #### kubectl expose deployment lamp-wp --name lamp-service --type NodePort --port 80 --target-port 80
    service/lamp-service exposed

  #### kubectl edit svc lamp-service
    # change the NodePort port
      ports:
      - nodePort: 32436
      ---
      ports:
      - nodePort: 30008
    =====
    service/lamp-service edited

  #### kubectl get all
    NAME                          READY   STATUS    RESTARTS   AGE
    pod/lamp-wp-d46d7f85c-tvsvf   2/2     Running   0          5m30s

    NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    service/kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP        53m
    service/lamp-service    NodePort    10.96.90.214   <none>        80:30008/TCP   3m8s
    service/mysql-service   ClusterIP   10.96.233.29   <none>        3306/TCP       2m28s

    NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/lamp-wp   1/1     1            1           5m30s

    NAME                                DESIRED   CURRENT   READY   AGE
    replicaset.apps/lamp-wp-d46d7f85c   1         1         1       5m30s

  #### kubectl exec lamp-wp-d46d7f85c-tvsvf -c httpd-php-container -- cat /opt/docker/etc/php/php.ini
    variables_order = "EGPCS"

  #### kubectl exec lamp-wp-d46d7f85c-tvsvf -c httpd-php-container -- env | grep MYSQL
    MYSQL_HOST=mysql-service
    MYSQL_ROOT_PASSWORD=rootpass123
    MYSQL_DATABASE=wpdb
    MYSQL_USER=wpuser
    MYSQL_PASSWORD=wppass123

  #### cp /tmp/index.php .
  - vi index.php 
  ```php
  <?php
  $dbname = getenv('MYSQL_DATABASE');
  $dbuser = getenv('MYSQL_USER');
  $dbpass = getenv('MYSQL_PASSWORD');
  $dbhost = getenv('MYSQL_HOST');

  $connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

  $test_query = "SHOW TABLES FROM $dbname";
  $result = mysqli_query($test_query);

  if ($result->connect_error) {
    die("Connection failed: " . $conn->connect_error);
  }
    echo "Connected successfully";
  ```

  #### kubectl cp index.php lamp-wp-d46d7f85c-tvsvf:/app -c httpd-php-container

  #### kubectl exec lamp-wp-d46d7f85c-tvsvf -c httpd-php-container -- ls /app
    index.php

  #### kubectl exec lamp-wp-d46d7f85c-tvsvf -c httpd-php-container -- cat /app/index.php
    <?php
    $dbname = getenv('MYSQL_DATABASE');
    $dbuser = getenv('MYSQL_USER');
    $dbpass = getenv('MYSQL_PASSWORD');
    $dbhost = getenv('MYSQL_HOST');

    $connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

    $test_query = "SHOW TABLES FROM $dbname";
    $result = mysqli_query($test_query);

    if ($result->connect_error) {
      die("Connection failed: " . $conn->connect_error);
    }
      echo "Connected successfully";

  #### curl localhost:8080
    Handling connection for 8080
    Connected successfully
</details>