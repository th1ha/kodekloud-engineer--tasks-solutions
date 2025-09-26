# Revert Deployment to Previous Version in Kubernetes

* There exists a deployment named `nginx-deployment`; initiate a rollback to the previous revision.

  ```bash
  kubectl get all
  
  kubectl rollout history deployment nginx-deployment

  kubectl rollout undo deployment nginx-deployment --to-revision=1

  ```
    <details>
    <summary>outputs</summary>

      # kubectl get all

        NAME                                    READY   STATUS    RESTARTS   AGE
        pod/nginx-deployment-698959d995-6gjjp   1/1     Running   0          19s
        pod/nginx-deployment-698959d995-9hnwp   1/1     Running   0          22s
        pod/nginx-deployment-698959d995-w8qw8   1/1     Running   0          17s

        NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
        service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        5m6s
        service/nginx-service   NodePort    10.96.217.169   <none>        80:30008/TCP   32s

        NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/nginx-deployment   3/3     3            3           32s

        NAME                                          DESIRED   CURRENT   READY   AGE
        replicaset.apps/nginx-deployment-698959d995   3         3         3       22s
        replicaset.apps/nginx-deployment-989f57c54    0         0         0       32s

      # kubectl rollout history deployment nginx-deployment

        deployment.apps/nginx-deployment 
        REVISION  CHANGE-CAUSE
        1         <none>
        2         kubectl set image deployment nginx-deployment nginx-container=nginx:alpine --kubeconfig=/root/.kube/config --record=true

      # kubectl rollout undo deployment nginx-deployment --to-revision=1

      deployment.apps/nginx-deployment rolled back

      # 
    </details>

---
* verify
    ```bash
    kubectl get all
    ```
    <details>
    <summary>verify-outputs</summary>

        # kubectl get all

        NAME                                    READY   STATUS              RESTARTS   AGE
        pod/nginx-deployment-698959d995-6gjjp   1/1     Running             0          3m29s
        pod/nginx-deployment-698959d995-9hnwp   1/1     Running             0          3m32s
        pod/nginx-deployment-698959d995-w8qw8   0/1     Terminating         0          3m27s
        pod/nginx-deployment-989f57c54-4j8q4    1/1     Running             0          2s
        pod/nginx-deployment-989f57c54-szxr4    0/1     ContainerCreating   0          1s

        NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
        service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        8m17s
        service/nginx-service   NodePort    10.96.217.169   <none>        80:30008/TCP   3m43s

        NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/nginx-deployment   3/3     2            3           3m43s

        NAME                                          DESIRED   CURRENT   READY   AGE
        replicaset.apps/nginx-deployment-698959d995   2         2         2       3m33s
        replicaset.apps/nginx-deployment-989f57c54    2         2         1       3m43s
        -----------
        NAME                                   READY   STATUS    RESTARTS   AGE
        pod/nginx-deployment-989f57c54-4j8q4   1/1     Running   0          3m3s
        pod/nginx-deployment-989f57c54-szxr4   1/1     Running   0          3m2s
        pod/nginx-deployment-989f57c54-wjthk   1/1     Running   0          3m

        NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
        service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        11m
        service/nginx-service   NodePort    10.96.217.169   <none>        80:30008/TCP   6m43s

        NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/nginx-deployment   3/3     3            3           6m43s

        NAME                                          DESIRED   CURRENT   READY   AGE
        replicaset.apps/nginx-deployment-698959d995   0         0         0       6m33s
        replicaset.apps/nginx-deployment-989f57c54    3         3         3       6m43s
    </details>