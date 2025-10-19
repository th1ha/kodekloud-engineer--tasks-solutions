# Fix Python App Deployed on Kubernetes Cluster

1. The deployment name is **python-deployment-devops**, its using **poroko/flask-demo-app**. The deployment and service of this app is already deployed.

2. nodePort should be **32345** and targetPort should be python flask app's default port.


---

<details>
<summary>steps</summary>

  #### kubectl get all
    NAME                                           READY   STATUS             RESTARTS   AGE
    pod/python-deployment-devops-678b746b7-wnn6k   0/1     ImagePullBackOff   0          79s

    NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    service/kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP          16m
    service/python-service-devops   NodePort    10.96.116.248   <none>        8080:32345/TCP   79s

    NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/python-deployment-devops   0/1     1            0           79s

    NAME                                                 DESIRED   CURRENT   READY   AGE
    replicaset.apps/python-deployment-devops-678b746b7   1         1         0       79s

  #### kubectl describe pod/python-deployment-devops-678b746b7-wnn6k
    Events:
    Type     Reason     Age                 From               Message
    ----     ------     ----                ----               -------
    Normal   Scheduled  108s                default-scheduler  Successfully assigned default/python-deployment-devops-678b746b7-wnn6k to kodekloud-control-plane
    Normal   Pulling    23s (x4 over 107s)  kubelet            Pulling image "poroko/flask-app-demo"
    Warning  Failed     23s (x4 over 107s)  kubelet            Failed to pull image "poroko/flask-app-demo": rpc error: code = Unknown desc = failed to pull and unpack image "docker.io/poroko/flask-app-demo:latest": failed to resolve reference "docker.io/poroko/flask-app-demo:latest": pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed
    Warning  Failed     23s (x4 over 107s)  kubelet            Error: ErrImagePull
    Normal   BackOff    10s (x5 over 106s)  kubelet            Back-off pulling image "poroko/flask-app-demo"
    Warning  Failed     10s (x5 over 106s)  kubelet            Error: ImagePullBackOff

  #### kubectl edit deployments.apps python-deployment-devops
    containers:
      - image: poroko/flask-app-demo
    ---
    containers:
      - image: poroko/flask-demo-app

  #### kubectl get all
    NAME                                            READY   STATUS    RESTARTS   AGE
    pod/python-deployment-devops-7859694dcf-shpq7   1/1     Running   0          18s

    NAME                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
    service/kubernetes              ClusterIP   10.96.0.1       <none>        443/TCP          19m
    service/python-service-devops   NodePort    10.96.116.248   <none>        8080:32345/TCP   4m7s

    NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/python-deployment-devops   1/1     1            1           4m7s

    NAME                                                  DESIRED   CURRENT   READY   AGE
    replicaset.apps/python-deployment-devops-678b746b7    0         0         0       4m7s
    replicaset.apps/python-deployment-devops-7859694dcf   1         1         1       18s

  #### kubectl get deployments.apps python-deployment-devops -oyaml | grep -i port
    ports:
    - containerPort: 5000

  #### kubectl get svc python-service-devops -oyaml | grep -i port
    ports:
    - nodePort: 32345
      port: 8080
      targetPort: 8080

  #### kubectl edit svc python-service-devops
    ports:
    - nodePort: 32345
      port: 8080
      protocol: TCP
      targetPort: 8080
    ---
    ports:
    - nodePort: 32345
      port: 5000
      protocol: TCP
      targetPort: 5000

  #### kubectl port-forward svc/python-service-devops 5000:5000 &
    [1] 3436
    Forwarding from 127.0.0.1:5000 -> 5000
    Forwarding from [::1]:5000 -> 5000
  
  #### curl localhost:5000
    Handling connection for 5000
    Hello World Pyvo 1!
</details>