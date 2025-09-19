# Execute Rolling Updates in Kubernetes

[Doc::: Rolling Update](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)

* Execute a rolling update for this application, integrating the `nginx:1.18` image. The deployment is named `nginx-deployment`

  ```bash
  kubectl get deployments.apps nginx-deployment -oyaml | grep -i -A3 "image:"
  
  kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18

  ```
    <details>
      <summary>outputs</summary>

      # kubectl get deployments.apps nginx-deployment -oyaml | grep -iE "image:|nginx:"

        - image: nginx:1.16
        imagePullPolicy: IfNotPresent
        name: nginx-container
        resources: {}

      # kubectl set image deployment/nginx-deployment nginx-container=nginx:1.18

      deployment.apps/nginx-deployment image updated

    </details>

---

* Verify

    ```bash
    kubectl get deployments.apps nginx-deployment -oyaml | grep -i -A3 "image:"

    ```
    <details>
      <summary>outputs</summary>

      # kubectl get deployments.apps nginx-deployment -oyaml | grep -iE "image:|nginx:"

        - image: nginx:1.18
        imagePullPolicy: IfNotPresent
        name: nginx-container
        resources: {}

    </details>