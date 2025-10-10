# Fix issue with LAMP Environment in Kubernetes

FYI, deployment name is **lamp-wp** and its using a service named **lamp-service**. The Apache is using http default port and nodeport is **30008**. From the application logs it has been identified that application is facing some issues while connecting to the database in addition to other issues. Additionally, there are some environment variables associated with the pods like **MYSQL_ROOT_PASSWORD, MYSQL_DATABASE,  MYSQL_USER, MYSQL_PASSWORD, MYSQL_HOST**.


> **Also do not try to delete/modify any other existing components like deployment name, service name, types, labels etc.**

<details>
<summary>details</summary>

  #### kubectl get all
    NAME                           READY   STATUS    RESTARTS   AGE
    pod/lamp-wp-56c7c454fc-xl6sl   2/2     Running   0          27s

    NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP          3m11s
    service/lamp-service    NodePort    10.96.64.204    <none>        8080:30008/TCP   27s
    service/mysql-service   ClusterIP   10.96.174.224   <none>        3306/TCP         27s

    NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/lamp-wp   1/1     1            1           27s

    NAME                                 DESIRED   CURRENT   READY   AGE
    replicaset.apps/lamp-wp-56c7c454fc   1         1         1       27s

  #### kubectl get svc lamp-service -oyaml | grep -i  port
    ports:
    - nodePort: 30008
      port: 8080
      targetPort: 8080
    type: NodePort

  #### kubectl get deployments.apps lamp-wp -oyaml  | grep -i -A2 port
    ports:
      - containerPort: 80
        name: httpd
        protocol: TCP

  #### kubectl edit svc lamp-service
  - change the targetPort port
    ports:
    - nodePort: 30008
      port: 8080
      protocol: TCP
      targetPort: 8080
    ---
    ports:
    - nodePort: 30008
      port: 8080
      protocol: TCP
      targetPort: 80
    =====
    service/lamp-service edited

  #### kubectl port-forward service/lamp-service 8080:8080 &
    [1] 3405
    Forwarding from [::1]:8080 -> 80

  #### curl localhost:8080
    Handling connection for 8080

  #### kubectl logs lamp-wp-56c7c454fc-xl6sl -c httpd-php-container
    [Fri Oct 10 03:53:26.150282 2025] [proxy_fcgi:error] [pid 126:tid 140407422335664] [client 127.0.0.1:51424] AH01071: Got error 'PHP message: PHP Parse error:  syntax error, unexpected 'MYSQL_PASSWORD' (T_STRING), expecting ']' in /app/index.php on line 4\n'
    127.0.0.1 - - [10/Oct/2025:03:53:26 +0000] "GET / HTTP/1.1" 500 - "-" "curl/7.76.1"
    [httpd:access] localhost:80 127.0.0.1 - - [10/Oct/2025:03:53:26 +0000] "GET / HTTP/1.1" 500 bytesIn:78 bytesOut:183 reqTime:0

  #### kubectl exec lamp-wp-56c7c454fc-xl6sl -c httpd-php-container -- cat app/index.php
    <?php
    $dbname = $_ENV['MYSQL_DATABASE'];
    $dbuser = $_ENV['MYSQL_USER'];
    $dbpass = $_ENV[''MYSQL_PASSWORD""];
    $dbhost = $_ENV['MYSQL-HOST'];


    $connect = mysqli_connect($dbhost, $dbuser, $dbpass) or die("Unable to Connect to '$dbhost'");

    $test_query = "SHOW TABLES FROM $dbname";
    $result = mysqli_query($test_query);

    if ($result->connect_error) {
      die("Connection failed: " . $conn->connect_error);
    }
      echo "Connected successfully";

  #### kubectl exec -it lamp-wp-56c7c454fc-xl6sl -c httpd-php-container -- env | grep -i MYSQL
    MYSQL_USER=kodekloud_gem
    MYSQL_PASSWORD=Rc5C9EyvbU
    MYSQL_HOST=mysql-service
    MYSQL_ROOT_PASSWORD=R00t
    MYSQL_DATABASE=kodekloud_db2
    MYSQL_SERVICE_SERVICE_PORT=3306
    MYSQL_SERVICE_PORT_3306_TCP_PROTO=tcp
    MYSQL_SERVICE_PORT_3306_TCP_ADDR=10.96.174.224
    MYSQL_SERVICE_PORT_3306_TCP_PORT=3306
    MYSQL_SERVICE_PORT_3306_TCP=tcp://10.96.174.224:3306
    MYSQL_SERVICE_SERVICE_HOST=10.96.174.224
    MYSQL_SERVICE_PORT=tcp://10.96.174.224:3306

  #### kubectl exec -it lamp-wp-56c7c454fc-xl6sl -c httpd-php-container -- sh
  ```sh
  cd app

  vi index.php

  $dbpass = $_ENV[''MYSQL_PASSWORD""];
  $dbhost = $_ENV['MYSQL-HOST'];
  ---
  $dbpass = $_ENV['MYSQL_PASSWORD'];  
  $dbhost = $_ENV['MYSQL_HOST'];

  ```

  #### curl localhost:8080
    Handling connection for 8080
    Connected successfully
</details>