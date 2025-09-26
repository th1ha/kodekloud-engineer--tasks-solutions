# Deploy Highly Available Pods with ReplicationController

[Doc:::](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#running-an-example-replicationcontroller)

* Create a `ReplicationController` using the `httpd` image with `latest` tag, and name it `httpd-replicationcontroller`
  - Assign labels `app` as `httpd_app`, and `type` as `front-end`. Ensure the container is named `httpd-container` and set the replica count to `3`
  - All `pods` should be running state post-deployment.

  <details>
    <summary>httpd-replica.yaml</summary>
      apiVersion: v1
      kind: ReplicationController
      metadata:
        name: httpd-replicationcontroller
      spec:
        replicas: 3
        selector:
          app: httpd_app
          type: front-end
        template:
          metadata:
            name: httpd-replicationcontroller
            labels:
              app: httpd_app
              type: front-end
          spec:
            containers:
            - name: httpd-container
              image: httpd:latest
              ports:
              - containerPort: 80
  </detauls>

---
* create & verify

  ```bash
  kubectl apply -f httpd-replica.yaml

  kubectl  get all

  kubectl get pods -l app=httpd_app,type=front-end

  kubectl get pods --show-labels

  ```
    <details>
      <summary>outputs</summary>

      # kubectl apply -f httpd-replica.yaml

      replicationcontroller/httpd-replicationcontroller created

      # kubectl  get all

      NAME                                    READY   STATUS    RESTARTS   AGE
      pod/httpd-replicationcontroller-4hwr9   1/1     Running   0          36s
      pod/httpd-replicationcontroller-98ht5   1/1     Running   0          36s
      pod/httpd-replicationcontroller-jwtd4   1/1     Running   0          36s

      NAME                                                DESIRED   CURRENT   READY   AGE
      replicationcontroller/httpd-replicationcontroller   3         3         3       36s

      # kubectl get pods -l app=httpd_app,type=front-end

      NAME                                READY   STATUS    RESTARTS   AGE
      httpd-replicationcontroller-4hwr9   1/1     Running   0          2m20s
      httpd-replicationcontroller-98ht5   1/1     Running   0          2m20s
      httpd-replicationcontroller-jwtd4   1/1     Running   0          2m20s

      # kubectl get pods --show-labels

      NAME                                READY   STATUS    RESTARTS   AGE     LABELS
      httpd-replicationcontroller-4hwr9   1/1     Running   0          3m36s   app=httpd_app,type=front-end
      httpd-replicationcontroller-98ht5   1/1     Running   0          3m36s   app=httpd_app,type=front-end
      httpd-replicationcontroller-jwtd4   1/1     Running   0          3m36s   app=httpd_app,type=front-end

    </details>