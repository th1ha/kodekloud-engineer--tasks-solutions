# Rolling Updates And Rolling Back Deployments in Kubernetes

* Create a namespace `devops`. Create a deployment called `httpd-deploy` under this new namespace, It should have one container called `httpd`, use `httpd:2.4.25` image and `3` replicas. The deployment should use `RollingUpdate` strategy with `maxSurge=1`, and `maxUnavailable=2`. Also create a `NodePort` type service named `httpd-service` and expose the deployment on `nodePort: 30008`
* Now upgrade the deployment to version `httpd:2.4.43` using a rolling update
* Finally, once all pods are updated undo the recent update and roll back to the previous/original version

[Doc - Rolling Update Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)

```bash
kubectl create ns devops

kubectl create deployment httpd-deploy --image=httpd:2.4.25 --replicas=3 --dry-run=client -oyaml > httpd-deploy.yaml

kubectl apply -f httpd-deploy.yaml

kubectl -n devops expose deployment httpd-deploy --name httpd-service --type NodePort --port 80 --target-port 80

# change NodePort
kubectl -n devops edit svc httpd-service

kubectl -n devops get all

kubectl -n devops port-forward svc/httpd-service 8080:80 &

curl -I localhost:8080

kubectl -n devops set image deployment.apps/httpd-deploy httpd=httpd:2.4.43

kubectl -n devops get all

curl -I localhost:8080

kubectl -n devops rollout history deployment httpd-deploy

kubectl -n devops rollout undo deployment httpd-deploy --to-revision=1

kubectl -n devops get all

curl -I localhost:8080
```
  <details>
  <summary>httpd-deploy.yaml</summary>
        
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    labels:
      app: httpd-deploy
    name: httpd-deploy
    namespace: devops
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: httpd-deploy
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxUnavailable: 2
        maxSurge: 1
    template:
      metadata:
        labels:
          app: httpd-deploy
      spec:
        containers:
        - image: httpd:2.4.25
          name: httpd
  ```
  </details>

  <details>
  <summary>outputs</summary>

  #### kubectl create ns devops
    namespace/devops created

  #### kubectl apply -f httpd-deploy.yaml
    deployment.apps/httpd-deploy created
  
  #### kubectl -n devops expose deployment httpd-deploy --name httpd-service --type NodePort --port 80 --target-port 80
    service/httpd-service exposed

  #### kubectl -n devops edit svc httpd-service
    ports:
    - nodePort: 32455 # old port
      port: 80
    ---
    ports:
    - nodePort: 30008 # new port
      port: 80

  #### kubectl -n devops get all
    NAME                                READY   STATUS    RESTARTS   AGE
    pod/httpd-deploy-6548d87dd8-2zwwb   1/1     Running   0          92s
    pod/httpd-deploy-6548d87dd8-6jrq6   1/1     Running   0          92s
    pod/httpd-deploy-6548d87dd8-dfgz8   1/1     Running   0          92s

    NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    service/httpd-service   NodePort   10.96.56.252   <none>        80:30008/TCP   39s

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/httpd-deploy   3/3     3            3           92s

    NAME                                      DESIRED   CURRENT   READY   AGE
    replicaset.apps/httpd-deploy-6548d87dd8   3         3         3       92s

  #### kubectl -n devops port-forward svc/httpd-service 8080:80 &
    [1] 4873
    Forwarding from [::1]:8080 -> 80

  #### curl -I localhost:8080
    Handling connection for 8080
    HTTP/1.1 200 OK
    Date: Wed, 01 Oct 2025 04:31:09 GMT
    Server: Apache/2.4.25 (Unix)

  #### kubectl -n devops set image deployment.apps/httpd-deploy httpd=httpd:2.4.43
    deployment.apps/httpd-deploy image updated

  #### kubectl -n devops get all
    NAME                                READY   STATUS    RESTARTS   AGE
    pod/httpd-deploy-6894dc88bb-8b2ck   1/1     Running   0          16s
    pod/httpd-deploy-6894dc88bb-ck8sq   1/1     Running   0          17s
    pod/httpd-deploy-6894dc88bb-tb8p8   1/1     Running   0          17s

    NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    service/httpd-service   NodePort   10.96.56.252   <none>        80:30008/TCP   2m46s

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/httpd-deploy   3/3     3            3           3m39s

    NAME                                      DESIRED   CURRENT   READY   AGE
    replicaset.apps/httpd-deploy-6548d87dd8   0         0         0       3m39s
    replicaset.apps/httpd-deploy-6894dc88bb   3         3         3       17s


  #### curl -I localhost:8080
    Handling connection for 8080
    HTTP/1.1 200 OK
    Date: Wed, 01 Oct 2025 04:37:45 GMT
    Server: Apache/2.4.43 (Unix)
    Last-Modified: Mon, 11 Jun 2007 18:53:14 GMT
    ETag: "2d-432a5e4a73a80"
    Accept-Ranges: bytes
    Content-Length: 45
    Content-Type: text/html
  
  #### kubectl -n devops rollout history deployment httpd-deploy
    deployment.apps/httpd-deploy 
    REVISION  CHANGE-CAUSE
    1         <none>
    2         <none>

  #### kubectl -n devops rollout undo deployment httpd-deploy --to-revision=1
    deployment.apps/httpd-deploy rolled back

  #### kubectl -n devops get all
    NAME                                READY   STATUS    RESTARTS   AGE
    pod/httpd-deploy-6548d87dd8-4b8zv   1/1     Running   0          94s
    pod/httpd-deploy-6548d87dd8-cwk5z   1/1     Running   0          95s
    pod/httpd-deploy-6548d87dd8-hpqbq   1/1     Running   0          94s

    NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
    service/httpd-service   NodePort   10.96.56.252   <none>        80:30008/TCP   5m44s

    NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/httpd-deploy   3/3     3            3           6m37s

    NAME                                      DESIRED   CURRENT   READY   AGE
    replicaset.apps/httpd-deploy-6548d87dd8   3         3         3       6m37s
    replicaset.apps/httpd-deploy-6894dc88bb   0         0         0       3m15s

  #### curl -I localhost:8080
    Handling connection for 8080
    HTTP/1.1 200 OK
    Date: Wed, 01 Oct 2025 04:55:22 GMT
    Server: Apache/2.4.25 (Unix)
  </details>