# Resolve Pod Deployment Issue

* There is a pod named `webserver`, and the container within it is named `httpd-container`, its utilizing the `httpd:latest` image.
    - Additionally, there's a sidecar container named `sidecar-container` using the `ubuntu:latest` image
    - Identify and address the issue to ensure the pod is in the `running` state and the application is accessible.


  ```bash
  kubectl  get all

  kubectl describe pod webserver

  kubectl edit pod webserver

  # removed the `s` from the image tag

  kubectl get all

  ```
    <details>
    <summary>outputs</summary>

      # kubectl  get all

      NAME            READY   STATUS         RESTARTS   AGE
      pod/webserver   1/2     ErrImagePull   0          72s

      NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
      service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        2m22s
      service/nginx-service   NodePort    10.96.127.178   <none>        80:30008/TCP   72s

      # kubectl describe pod webserver

      Warning  Failed     107s (x4 over 3m19s)   kubelet            Failed to pull image "httpd:latests": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/library/httpd:latests": failed to resolve reference "docker.io/library/httpd:latests": docker.io/library/httpd:latests: not found

      # kubectl get all

      NAME            READY   STATUS    RESTARTS   AGE
      pod/webserver   2/2     Running   0          5m25s

      NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
      service/kubernetes      ClusterIP   10.96.0.1       <none>        443/TCP        6m35s
      service/nginx-service   NodePort    10.96.127.178   <none>        80:30008/TCP   5m25s

    </details>