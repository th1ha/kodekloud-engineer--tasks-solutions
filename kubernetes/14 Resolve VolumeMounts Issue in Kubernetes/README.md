#  Resolve VolumeMounts Issue in Kubernetes

* The pod name is `nginx-phpfpm` and configmap name is `nginx-config`. Identify and fix the problem
  - Once resolved, copy `/home/thor/index.php` file from the `jump host` to the `nginx-container` within the nginx document root. After this, you should be able to access the website using `Website` button on the top bar


---
* Steps

  ```bash
  kubectl  get cm

  kubectl  get all

  kubectl  get cm nginx-config -oyaml

  kubectl get pod nginx-phpfpm -oyaml | grep -i "html"

  kubectl edit pod nginx-phpfpm

  kubectl apply -f /tmp/kubectl-edit-2507514645.yaml --force

  kubectl  get po

  kubectl cp index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container

  # click the Website buttom

  ```
    <details>
      <summary>cm - kubectl  get cm nginx-config -oyaml</summary>
        
      apiVersion: v1
      data:
        nginx.conf: |
          events {
          }
          http {
            server {
              listen 8099 default_server;
              listen [::]:8099 default_server;

              # Set nginx to serve files from the shared volume!
              root /var/www/html;
              index  index.html index.htm index.php;
              server_name _;
              location / {
                try_files $uri $uri/ =404;
              }
              location ~ \.php$ {
                include fastcgi_params;
                fastcgi_param REQUEST_METHOD $request_method;
                fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                fastcgi_pass 127.0.0.1:9000;
              }
            }
          }
      kind: ConfigMap
      metadata:
        annotations:
          kubectl.kubernetes.io/last-applied-configuration: |
            {"apiVersion":"v1","data":{"nginx.conf":"events {\n}\nhttp {\n  server {\n    listen 8099 default_server;\n    listen [::]:8099 default_server;\n\n    # Set nginx to serve files from the shared volume!\n    root /var/www/html;\n    index  index.html index.htm index.php;\n    server_name _;\n    location / {\n      try_files $uri $uri/ =404;\n    }\n    location ~ \\.php$ {\n      include fastcgi_params;\n      fastcgi_param REQUEST_METHOD $request_method;\n      fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;\n      fastcgi_pass 127.0.0.1:9000;\n    }\n  }\n}\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"nginx-config","namespace":"default"}}
        creationTimestamp: "2025-09-26T04:11:03Z"
        name: nginx-config
        namespace: default
        resourceVersion: "980"
        uid: 14bf365f-c4fa-4a4f-8e3b-5434404e31aa

    </details>

    <details>
      <summary>outputs</summary>

      # kubectl  get cm

      NAME               DATA   AGE
      kube-root-ca.crt   1      9m42s
      nginx-config       1      3m21s

      # kubectl  get all

      NAME               READY   STATUS    RESTARTS   AGE
      pod/nginx-phpfpm   2/2     Running   0          4m18s

      NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
      service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP          14m
      service/nginx-service   NodePort    10.96.169.187   <none>        8099:30008/TCP   4m18s

      # kubectl get pod nginx-phpfpm -oyaml | grep -i "html"

      - mountPath: /usr/share/nginx/html
      - mountPath: /var/www/html

      # kubectl edit pod nginx-phpfpm

      From/
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: shared-files
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-rbbcj
          readOnly: true
      To/
        volumeMounts:
        - mountPath: /var/www/html
          name: shared-files
        - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
          name: kube-api-access-rbbcj
          readOnly: true

      error: pods "nginx-phpfpm" is invalid
      A copy of your changes has been stored to "/tmp/kubectl-edit-2507514645.yaml"
      error: Edit cancelled, no valid changes were saved.

      # kubectl apply -f /tmp/kubectl-edit-2507514645.yaml --force

      pod/nginx-phpfpm configured

      # kubectl  get po

      nginx-phpfpm   2/2     Running   0          60s

      # kubectl cp index.php nginx-phpfpm:/var/www/html/index.php -c nginx-container
      
    </details>