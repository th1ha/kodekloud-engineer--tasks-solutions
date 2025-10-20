# Deploy Redis Deployment on Kubernetes

1. Create a **config map** called **my-redis-config** having **maxmemory 2mb** in **redis-config**.

2. Name of the **deployment** should be **redis-deployment**, it should use
**redis:alpine** image and container name should be **redis-container**. Also make sure it has only **1** replica.

3. The container should request for **1** CPU.

4. Mount **2** volumes:
  - An Empty directory volume called **data** at path **/redis-master-data**.
  - A configmap volume called **redis-config** at path **/redis-master**.
  - The container should expose the port **6379**.

5. Finally, **redis-deployment** should be in an up and running state.

- [Doc - pod resources example](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#example-2)
- [Doc - configMap example](https://kubernetes.io/docs/concepts/storage/volumes/#configmap)
---

<details>
<summary>steps</summary>

  #### create a configmap
  ```bash
  kubectl create configmap my-redis-config --from-literal=redis-config='maxmemory 2mb'
  ```

  #### Create a deployment
  ```bash
  kubectl create deployment redis-deployment --image redis:alpine --replicas 1 --port 6379 --dry-run=client -oyaml > redis.yaml
  ```
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    creationTimestamp: null
    labels:
      app: redis-deployment
    name: redis-deployment
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: redis-deployment
    strategy: {}
    template:
      metadata:
        creationTimestamp: null
        labels:
          app: redis-deployment
      spec:
        volumes:
          - name: data
            emptyDir: {}
          - name: redis-config
            configMap:
              name: my-redis-config
        containers:
        - image: redis:alpine
          name: redis-container
          ports:
          - containerPort: 6379
          resources:
            requests:
              cpu: "1"
          volumeMounts:
            - name: data
              mountPath: /redis-master-data
            - name: redis-config
              mountPath: /redis-master
  ```

  #### kubectl apply -f redis.yaml 
    deployment.apps/redis-deployment created

  #### kubectl get all
    NAME                                    READY   STATUS    RESTARTS   AGE
    pod/redis-deployment-7bccd99d85-fmrsb   1/1     Running   0          26s

    NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
    service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   14m

    NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/redis-deployment   1/1     1            1           26s

    NAME                                          DESIRED   CURRENT   READY   AGE
    replicaset.apps/redis-deployment-7bccd99d85   1         1         1       26s

  #### kubectl exec redis-deployment-7bccd99d85-fmrsb -- ls /redis-master
    redis-config

  #### kubectl exec redis-deployment-7bccd99d85-fmrsb -- cat /redis-master/redis-config
    maxmemory 2mb
</details>