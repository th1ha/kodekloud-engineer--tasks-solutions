# Deploy Nginx Web Server on Kubernetes Cluster

* Create a deployment using `nginx` image with `latest` tag only and remember to mention the tag i.e `nginx:latest`. Name it as `nginx-deployment`. The container should be named as `nginx-container`, also make sure replica counts are `3`
* Create a `NodePort` type service named `nginx-service`. The nodePort should be `30011`

```bash
kubectl create deployment nginx-deployment --image nginx:latest --replicas=3 --dry-run=client -oyaml > nginx-deployment.yaml

# change the container name
vi nginx-deployment.yaml

kubectl  apply -f nginx-deployment.yaml

kubectl expose deployment nginx-deployment --type=NodePort --name=nginx-service --target-port=80 --port=80 --dry-run=client -oyaml > svc.yaml

# add nodePort: 30011
vi svc.yaml

kubectl  apply -f svc.yaml

# testing
kubectl port-forward service/nginx-service 30011:80 &

curl localhost:30011

```
  <details>
  <summary>nginx-deployment.yaml</summary>
        
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: nginx-deployment
    name: nginx-deployment
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx-deployment
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: nginx-deployment
      spec:
        containers:
        - image: nginx:latest
          name: nginx-container
  ```
  </details>

  <details>
  <summary>svc.yaml</summary>
        
  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    creationTimestamp: null
    labels:
      app: nginx-deployment
    name: nginx-service
  spec:
    ports:
    - port: 80
      protocol: TCP
      targetPort: 80
      nodePort: 30011
    selector:
      app: nginx-deployment
    type: NodePort
  ```
  </details>

  <details>
  <summary>outputs</summary>

    # kubectl create deployment nginx-deployment --image nginx:latest --replicas=3 --dry-run=client -oyaml > nginx-deployment.yaml

    # kubectl  apply -f nginx-deployment.yaml

    deployment.apps/nginx-deployment created

    # kubectl expose deployment nginx-deployment --type=NodePort --name=nginx-service --target-port=80 --port=80 --dry-run=client -oyaml > svc.yaml

    # kubectl  apply -f svc.yaml

    service/nginx-service created

    # kubectl port-forward service/nginx-service 30011:80 &

    Forwarding from [::1]:30011 -> 80

    # curl localhost:30011
      Handling connection for 30011
      <!DOCTYPE html>
      <html>
      <head>
      <title>Welcome to nginx!</title>
      <style>
      html { color-scheme: light dark; }
      body { width: 35em; margin: 0 auto;
      font-family: Tahoma, Verdana, Arial, sans-serif; }
      </style>
      </head>
      <body>
      <h1>Welcome to nginx!</h1>
      <p>If you see this page, the nginx web server is successfully installed and
      working. Further configuration is required.</p>

      <p>For online documentation and support please refer to
      <a href="http://nginx.org/">nginx.org</a>.<br/>
      Commercial support is available at
      <a href="http://nginx.com/">nginx.com</a>.</p>

      <p><em>Thank you for using nginx.</em></p>
      </body>
      </html>

  </details>