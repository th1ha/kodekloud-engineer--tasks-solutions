# Kubernetes LEMP Setup

1. Create some secrets for MySQL.
  - Create a secret named **mysql-root-pass** wih key/value pairs as below:
    ```
    name: password
    value: R00t
    ```

  - Create a secret named **mysql-user-pass** with key/value pairs as below:
    ```
    name: username
    value: kodekloud_gem

    name: password
    value: ksH85UJjhb
    ```

  - Create a secret named **mysql-db-url** with key/value pairs as below:
    ```
    name: database
    value: kodekloud_db6
    ```

  - Create a secret named **mysql-host** with key/value pairs as below:
    ```
    name: host
    value: mysql-service
    ```

2. Create a config map **php-config** for **php.ini** with **variables_order = "EGPCS"** data.

3. Create a deployment named **lemp-wp**.

4. Create two containers under it. First container must be **nginx-php-container** using image **webdevops/php-nginx:alpine-3-php7** and second container must be **mysql-container** from image **mysql:5.6**. Mount **php-config** configmap in nginx container at **/opt/docker/etc/php/php.ini** location.

5. Add some environment variables for both containers:
    - **MYSQL_ROOT_PASSWORD**, **MYSQL_DATABASE**, **MYSQL_USER**, **MYSQL_PASSWORD** and **MYSQL_HOST**. Take their values from the secrets you created. Please make sure to use env field (do not use envFrom) to define the name-value pair of environment variables.

6. Create a node port type service **lemp-service** to expose the web application, nodePort must be **30008**.

7. Create a service for mysql named **mysql-service** and its port must be **3306**.

8. We already have a **/tmp/index.php** file on **jump_host** server.
    - Copy this file into the **nginx** container under document root i.e **/app** and replace the dummy values for mysql related variables with the environment variables you have set for mysql related parameters. Please make sure you do not hard code the mysql related details in this file, you must use the environment variables to fetch those values.
    - Once done, you must be able to access this website using **Website** button on the top bar, please note that you should see **Connected successfully** message while accessing this page.

- [Doc - Using ConfigMaps as files from a Pod ](https://kubernetes.io/docs/concepts/configuration/configmap/#using-configmaps-as-files-from-a-pod)
- [Doc - Using subPath](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath)
- [Doc - container environment variable with data from secret](https://kubernetes.io/docs/concepts/storage/volumes/#using-subpath)


<details>
<summary>steps</summary>

  #### Create secrets for MYSQL
  ```bash
  kubectl create secret generic mysql-root-pass --from-literal=password=R00t
  # secret/mysql-root-pass created

  kubectl create secret generic mysql-user-pass --from-literal=username=kodekloud_gem --from-literal=password=ksH85UJjhb
  # secret/mysql-user-pass created

  kubectl create secret generic mysql-db-url --from-literal=database=kodekloud_db6
  # secret/mysql-db-url created

  kubectl create secret generic mysql-host --from-literal=host=mysql-service
  # secret/mysql-host created
  ```

  #### create a config map
  ```bash
  kubectl create cm php-config --from-literal="php.ini"='variables_order = "EGPCS"'
  # configmap/php-config created
  ```

  #### create a deployment template
  ```bash
  kubectl create deployment lemp-wp --image webdevops/php-nginx:alpine-3-php7 --image mysql:5.6 --port 80 --dry-run=client -oyaml > lemp-wp.yaml

  kubectl apply -f lemp-wp.yaml 
  # deployment.apps/lemp-wp created
  ```
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: lemp-wp
    name: lemp-wp
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: lemp-wp
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: lemp-wp
      spec:
        volumes:
        - name: php-config
          configMap:
            name: php-config
        containers:
        - image: webdevops/php-nginx:alpine-3-php7
          name: nginx-php-container
          ports:
          - containerPort: 80
          volumeMounts:
          - name: php-config
            mountPath: /opt/docker/etc/php/php.ini
            subPath: php.ini
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
          - name: MYSQL_HOST
            valueFrom:
              secretKeyRef:
                name: mysql-host
                key: host
        - image: mysql:5.6
          name: mysql-container
          ports:
          - containerPort: 3306
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
          - name: MYSQL_HOST
            valueFrom:
              secretKeyRef:
                name: mysql-host
                key: host
  ```

  #### create a service
  ```bash
  kubectl expose deployment lemp-wp --name lemp-service --type NodePort --port 80 --target-port 80
  #service/lemp-service exposed
  
  kubectl expose deployment lemp-wp --name mysql-service --port 3306 --target-port 3306
  #service/mysql-service exposed
  ```

  #### change the NodePort port
  ```bash
  kubectl edit svc lemp-service
  ```
  ```
  ports:
  - nodePort: 31376
  ---
  ports:
  - nodePort: 30008
  ```

  #### testing
  ```bash
  kubectl port-forward svc/lemp-service 8080:80 &
  curl localhost:8080
  ```
  - output
    ```
    Handling connection for 8080
    <html>
    <head><title>403 Forbidden</title></head>
    <body bgcolor="white">
    <center><h1>403 Forbidden</h1></center>
    <hr><center>nginx/1.10.3</center>
    </body>
    </html>
    ```

  #### copy index.php to nginx container
  ```bash
  cp /tmp/index.php .

  kubectl cp index.php lemp-wp-78cc4bf44c-b94m8:/app -c nginx-php-container

  kubectl exec lemp-wp-78cc4bf44c-b94m8 -c nginx-php-container -- ls /app
  # index.php
  ```
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
  #### testing
  ```bash
  curl localhost:8080
  ```
  - output
    ```
    Handling connection for 8080
    Connected successfully
    ```
</details>