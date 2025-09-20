# Deploy ReplicaSet in Kubernetes Cluster

* Create a ReplicaSet using `httpd` image with `latest` tag (ensure to specify as `httpd:latest`) and name it `httpd-replicaset`
    - Apply labels: `app` as `httpd_app`, `type` as `front-end`
    - Name the container `httpd-container`. Ensure the replica count is `4`

  ```bash
  kubectl create deployment httpd-replicaset --image=httpd:latest --replicas=4 --dry-run=client -oyaml > httpd.yaml

  ```
    <details>
    <summary>httpd.yaml</summary>

      # vim httpd.yaml - Change from Deployment to ReplicaSet
      apiVersion: apps/v1
      kind: ReplicaSet
      metadata:
      labels:
          app: httpd_app
          type: front-end
      name: httpd-replicaset
      spec:
      replicas: 4
      selector:
          matchLabels:
          app: httpd_app
          type: front-end
      template:
          metadata:
          labels:
              app: httpd_app
              type: front-end
          spec:
          containers:
          - image: httpd:latest
              name: httpd-container

    </details>

---
* apply and verify

  ```bash
  kubectl apply -f httpd.yaml

  kubectl  get all

  kubectl get all --label-columns=app=httpd_app
  ```
    <details>
    <summary>outputs</summary>

      # kubectl apply -f httpd.yaml
      
      replicaset.apps/httpd-replicaset created
      -----
      # kubectl get all
      
      NAME                         READY   STATUS    RESTARTS   AGE
      pod/httpd-replicaset-djjkv   1/1     Running   0          34s
      pod/httpd-replicaset-fzgjf   1/1     Running   0          34s
      pod/httpd-replicaset-pb7m5   1/1     Running   0          34s
      pod/httpd-replicaset-vlzpz   1/1     Running   0          34s

      NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
      service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   28m

      NAME                               DESIRED   CURRENT   READY   AGE
      replicaset.apps/httpd-replicaset   4         4         4       34s
      -----
      # kubectl get all --label-columns=app=httpd_app
      
      NAME                         READY   STATUS    RESTARTS   AGE     APP=HTTPD_APP
      pod/httpd-replicaset-djjkv   1/1     Running   0          3m27s   
      pod/httpd-replicaset-fzgjf   1/1     Running   0          3m27s   
      pod/httpd-replicaset-pb7m5   1/1     Running   0          3m27s   
      pod/httpd-replicaset-vlzpz   1/1     Running   0          3m27s   

      NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE   APP=HTTPD_APP
      service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   31m   

      NAME                               DESIRED   CURRENT   READY   AGE     APP=HTTPD_APP
      replicaset.apps/httpd-replicaset   4         4         4       3m27s   

    </details>