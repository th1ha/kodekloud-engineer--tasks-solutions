# Kubernetes Nginx and PhpFPM Setup

1. Create a service to expose this app, the service type must be **NodePort**, nodePort should be **30012**.

2. Create a config map named **nginx-config** for **nginx.conf** as we want to add some custom settings in nginx.conf.
  - Change the default port **80** to **8098** in **nginx.conf**.
  - Change the default document root **/usr/share/nginx** to **/var/www/html** in **nginx.conf**.
  - Update the directory index to index  **index.html index.htm index.php** in **nginx.conf**.

3. Create a pod named **nginx-phpfpm** .
  - Create a shared volume named **shared-files** that will be used by both containers (nginx and phpfpm) also it should be a **emptyDir** volume.
  - Map the ConfigMap we declared above as a volume for nginx container. Name the volume as **nginx-config-volume**, mount path should be **/etc/nginx/nginx.conf** and subPath should be **nginx.conf**
  - Nginx container should be named as **nginx-container** and it should use **nginx:latest** image. PhpFPM container should be named as **php-fpm-container** and it should use **php:8.2-fpm-alpine** image.
  - The shared volume **shared-files** should be mounted at **/var/www/html** location in both containers. Copy **/opt/index.php** from **jump host** to the nginx document root inside the **nginx** container, once done you can access the app using **App** button on the top bar.

Before clicking on **finish** button always make sure to check if all pods are in **running** state.

> You can use any labels as per your choice.
---


<details>
<summary>svc.yaml</summary>

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx-phpfpm
spec:
  type: NodePort
  selector:
    app: nginx-phpfpm
  ports:
    - port: 8098
      targetPort: 8098
      nodePort: 30012
```
</details>

<details>
<summary>cm.yaml</summary>

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    http {
      server {
        listen 8098;
        root /var/www/html;
        index index.html index.htm index.php;
        location / {
          try_files $uri $uri/ =404;
        }
      }
    }
```
</details>

<details>
<summary>pod.yaml</summary>

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-phpfpm
  labels:
    app: nginx-phpfpm
spec:
  volumes:
    - name: shared-files
      emptyDir: {}
    - name: nginx-config-volume
      configMap:
        name: nginx-config
  containers:
    - name: nginx-container
      image: nginx:latest
      volumeMounts:
        - name: shared-files
          mountPath: /var/www/html
        - name: nginx-config-volume
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
      ports:
        - containerPort: 8098
    - name: php-fpm-container
      image: php:8.2-fpm-alpine
      volumeMounts:
        - name: shared-files
          mountPath: /var/www/html

```
</details>

<details>
<summary>steps</summary>

  #### create all manifest files and copy the PHP file
  ```bash
  kubectl apply -f cm.yaml -f pod.yaml -f svc.yaml

  kubectl cp /opt/index.php nginx-phpfpm:/var/www/html -c nginx-container
  ```

  #### Testing
  ```bash
  kubectl port-forward service/nginx-service 8080:8098 &

  curl localhost:8080
  ```
</details>