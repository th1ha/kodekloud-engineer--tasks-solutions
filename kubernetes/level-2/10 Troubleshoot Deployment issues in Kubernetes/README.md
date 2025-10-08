# Troubleshoot Deployment issues in Kubernetes

> The deployment name is **redis-deployment**. The pods are not in running state right now, so please look into the issue and fix the same.

```bash
kubectl  get all

kubectl describe pod/redis-deployment-6fd9d5fcb-szphg

kubectl get cm

kubectl edit deployments.apps redis-deployment

kubectl get all

kubectl describe pod/redis-deployment-5bcd4c7d64-wtxzz

kubectl edit deployments.apps redis-deployment

kubectl get all
```

<details>
<summary>outputs</summary>

  #### kubectl  get all
    NAME                                   READY   STATUS              RESTARTS   AGE
    pod/redis-deployment-6fd9d5fcb-szphg   0/1     ContainerCreating   0          112s

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   9m5s

    NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/redis-deployment   0/1     1            0           112s

    NAME                                         DESIRED   CURRENT   READY   AGE
    replicaset.apps/redis-deployment-6fd9d5fcb   1         1         0       112s

  #### kubectl describe pod/redis-deployment-6fd9d5fcb-szphg
    Events:
    Type     Reason       Age                  From               Message
    ----     ------       ----                 ----               -------
    Normal   Scheduled    2m33s                default-scheduler  Successfully assigned default/redis-deployment-6fd9d5fcb-szphg to kodekloud-control-plane
    Warning  FailedMount  30s                  kubelet            Unable to attach or mount volumes: unmounted volumes=[config], unattached volumes=[], failed to process volumes=[]: timed out waiting for the condition
    Warning  FailedMount  25s (x9 over 2m33s)  kubelet            MountVolume.SetUp failed for volume "config" : configmap "redis-cofig" not found

  #### kubectl get cm
    NAME               DATA   AGE
    kube-root-ca.crt   1      10m
    redis-config       2      3m5s

  #### kubectl edit deployments.apps redis-deployment
    - configMap:
        defaultMode: 420
        name: redis-cofig
      name: config
    ---
    - configMap:
        defaultMode: 420
        name: redis-config
      name: config
    =====
    deployment.apps/redis-deployment edited

  #### kubectl get all
    NAME                                    READY   STATUS              RESTARTS   AGE
    pod/redis-deployment-5bcd4c7d64-wtxzz   0/1     ErrImagePull        0          8s
    pod/redis-deployment-6fd9d5fcb-szphg    0/1     ContainerCreating   0          5m39s

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   12m

    NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/redis-deployment   0/1     1            0           5m39s

    NAME                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/redis-deployment-5bcd4c7d64   1         1         0       8s
    replicaset.apps/redis-deployment-6fd9d5fcb    1         1         0       5m39s

  #### kubectl describe pod/redis-deployment-5bcd4c7d64-wtxzz
    Events:
    Type     Reason     Age                From               Message
    ----     ------     ----               ----               -------
    Normal   Scheduled  69s                default-scheduler  Successfully assigned default/redis-deployment-5bcd4c7d64-wtxzz to kodekloud-control-plane
    Normal   Pulling    29s (x3 over 68s)  kubelet            Pulling image "redis:alpin"
    Warning  Failed     28s (x3 over 68s)  kubelet            Failed to pull image "redis:alpin": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/redis:alpin": failed to resolve reference "docker.io/library/redis:alpin": docker.io/library/redis:alpin: not found
    Warning  Failed     28s (x3 over 68s)  kubelet            Error: ErrImagePull
    Normal   BackOff    4s (x4 over 67s)   kubelet            Back-off pulling image "redis:alpin"
    Warning  Failed     4s (x4 over 67s)   kubelet            Error: ImagePullBackOff

  #### kubectl edit deployments.apps redis-deployment
    spec:
      containers:
      - image: redis:alpin
    ---
    spec:
      containers:
      - image: redis:alpine
    =====
    deployment.apps/redis-deployment edited

  #### kubectl get all
    NAME                                    READY   STATUS        RESTARTS   AGE
    pod/redis-deployment-6fd9d5fcb-szphg    0/1     Terminating   0          8m9s
    pod/redis-deployment-7c8d4f6ddf-fmv6c   1/1     Running       0          13s

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15m

    NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/redis-deployment   1/1     1            1           8m9s

    NAME                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/redis-deployment-5bcd4c7d64   0         0         0       2m38s
    replicaset.apps/redis-deployment-6fd9d5fcb    0         0         0       8m9s
    replicaset.apps/redis-deployment-7c8d4f6ddf   1         1         1       13s
</details>