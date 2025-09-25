# Update Deployment and Service in Kubernetes

* Modify the service nodeport from `30008` to `32165`
  - Change the replicas count from `1` to `5`
  - Update the image from `nginx:1.19` to `nginx:latest`


  ```bash
  kubectl get svc

  kubectl edit svc nginx-service 

  # verify
  kubectl get svc

  kubectl  get deployments.apps

  kubectl scale deployment nginx-deployment --replicas 5

  # verify
  kubectl  get deployments.apps

  # check the current image
  kubectl get deployments.apps nginx-deployment -oyaml | grep -i -C2 "image: nginx:"

  # kubectl set image deployment/nginx-deployment nginx-container=nginx:latest

  # verify
  kubectl get deployments.apps nginx-deployment -oyaml | grep -i -C2 "image: nginx:"

  ```
    <details>
    <summary>outputs</summary>

      # kubectl get svc

      NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
      kubernetes      ClusterIP   10.96.0.1     <none>        443/TCP        8m14s
      nginx-service   NodePort    10.96.61.22   <none>        80:30008/TCP   2m18s

      # kubectl edit svc nginx-service

      ports:
      - nodePort: 32165

      # kubectl get svc

      NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
      kubernetes      ClusterIP   10.96.0.1     <none>        443/TCP        10m
      nginx-service   NodePort    10.96.61.22   <none>        80:32165/TCP   4m33s

      # kubectl  get deployments.apps

      NAME               READY   UP-TO-DATE   AVAILABLE   AGE
      nginx-deployment   1/1     1            1           5m24s

      # kubectl scale deployment nginx-deployment --replicas 5

      deployment.apps/nginx-deployment scaled

      # kubectl  get deployments.apps

      NAME               READY   UP-TO-DATE   AVAILABLE   AGE
      nginx-deployment   5/5     5            5           7m2s

      # kubectl get deployments.apps nginx-deployment -oyaml | grep -i -C2 "image: nginx:"

      spec:
      containers:
      - image: nginx:1.19
        imagePullPolicy: IfNotPresent
        name: nginx-container

      # kubectl set image deployment/nginx-deployment nginx-container=nginx:latest

      deployment.apps/nginx-deployment image updated

      # kubectl get deployments.apps nginx-deployment -oyaml | grep -i -C2 "image: nginx:"

      spec:
      containers:
      - image: nginx:latest
        imagePullPolicy: IfNotPresent
        name: nginx-container

    </details>